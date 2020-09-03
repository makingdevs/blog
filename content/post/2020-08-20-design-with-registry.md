---
title: "Diseño con procesos, usando Registry"
author: "neodevelop"
type: "post"
date: 2020-08-20T17:09:15-05:00
subtitle: "¿Cómo construimos aplicaciones con OTP?"
tags: ["elixir","otp", "domino", "design"]
---

Hace tiempo hemos estado construyendo un juego de [Domino](https://domino.makingdevs.com), basándonos sólo en el ecosistema proveído por Erlang/Elixir, y lo que deseo explicar aquí es una segunda parte de los elementos que nos ayudaron a diseñarlo, sólo usando elementos basados en OTP, aquí explico otro de ellos.

Aquí puedes encontrar la primer parte: [Agentes y supervisores en OTP]({{< ref "/post/2020-07-27-agent-and-supervisor-otp.md" >}} "OTP")

# ETS

Cuando estamos creando procesos de forma dinámica, queremos almacenar una referencia de ellos en alguna parte, y para ello nos sirve [`:ets`][ets]; solamente recordar que no se recomienda usar [ETS][ets] cómo un cache de forma prematura.

[ETS][ets] nos permite almacenar valores de Elixir en memoria, para su rápido acceso; en breve los datos son organizados en un conjunto de tablas dinámicas, las cuáles almacenan tuplas. Cada tabla es creada por un proceso y cuándo este termina entonces la tabla se destruye.

Existen tipos de tablas y la interface le pertenece a Erlang, sin embargo, ejemplos de este estilo ya vienen en la documentación de Elixir y Erlang, de hecho hay ejemplos de implementaciones de caché y de almacenes con ETS, pero hay una nota al final que sugiere el uso de Registry para dichas tareas.

# Registry

Usaré como referencia la documentación de [`Registry`][registry] en Elixir, diciendo que:

> Es un almacén de procesos de llave valor, local, decentralizado y escalable.

Se pueden tener registros de procesos únicos(`:unique`) o duplicados(`:duplicate`). Adicionalmente, Registry usa ETS internamente, en total 3 tablas, una para registros y dos más para particiones.

- `Elixir.Registry.____`
- `Elixir.Registry.____.KeyPartition0`
- `Elixir.Registry.____.PIDPartition0`

En dónde `____` es el nombre del átomo que le asignas al registro.

## Usando :via

Lo que se necesita para registrar un proceso es una tupla de la forma:

> `{:via, Registry, {registry, key}}`

En dónde `registry` es el nombre del átomo de registro personalizado y `key` es un identificador asignado de forma arbitraria.

Y para buscar un proceso sería suficiente utilizar la función `Registry.lookup(atom, id)`, en dónde `atom` es el nombre del registro y el parámetro `id` es el identificador con el que se registro el proceso. Por ejemplo: `Registry.lookup(Registry.ViaGame, name)`.

## Primer abstracción

Usaré el registro para almacenar procesos con el comportamiento de `Agent`:

```elixir
defmodule TheLiveCounter.Game do
  alias __MODULE__
  use Agent, restart: :temporary
  defstruct counter: 0, id: 0, name: ""

  ##  API Client

  def start_link(opts \\ []) do
    [name: name] = opts
    Agent.start_link(fn -> create(name) end, name: via_tuple(name))
  end

  defp create(name) do
    %Game{id: System.unique_integer([:positive]), name: name}
  end

  def get_counter(name) do
    Agent.get(via_tuple(name), fn %Game{counter: counter} -> counter end)
  end

  def get(name) do
    Agent.get(via_tuple(name), & &1)
  end

  def increase_counter(name) do
    Agent.update(via_tuple(name), fn %Game{counter: counter} = game ->
      %Game{game | counter: counter + 1}
    end)
  end

  defp via_tuple(name) do
    {:via, Registry, {Registry.ViaGame, name}}
  end
end
```

Lo que quiero resaltar aquí es la función privada `via_tuple`, la cuál me ayuda a generar la tupla necesaria para el uso de `Registry`, el cuál hago en `start_link`, dónde sólo busco un nombre para registrar, dado ese nombre entonces puedo operar las funciones del módulo `Game`, y sin exponer explicítamente el uso de procesos estoy operando sobre aquellos procesos registrados, esto en cada función del cliente: `get`, `increase_counter`.

## Administrador del Registry

Agregué un `GenServer` que me ayudó a supervisar el proceso que creaba los agentes, aquí resumo el código para detallarlo:

```elixir
defmodule TheLiveCounter.GameRegistry do
  alias TheLiveCounter.Game
  use GenServer

  @supervisor TheLiveCounter.DynamicSupervisor

  ## Client API

  def start_link(opts \\ []) do
    GenServer.start_link(__MODULE__, [], [name: __MODULE__] ++ opts)
  end

  def create do
    GenServer.call(__MODULE__, {:create})
  end

  def lookup(name) do
    case Registry.lookup(Registry.ViaGame, name) do
      [{pid_game, _}] -> pid_game
      [] -> nil
    end
  end

  ## More client functions...

  ##  GenServer Callbacks

  def handle_call({:create}, _from, state) do
    name = random_name()
    {:ok, game_pid} = DynamicSupervisor.start_child(@supervisor, {Game, name: name})
    {:reply, {:ok, name, game_pid}, state}
  end

  defp random_name() do
    ?a..?z
    |> Enum.take_random(6)
    |> List.to_string()
  end

  ## More callbacks...
end
```

El callback que recibe `{:create}` usará un Supervisor dinámico al cuál lo parametrizamos envíandole un nombre aleatorio, que ya recibe por sí mismo el comportamiento del Agente `Game`.

## Supervisando Registry

Finalmente supervisamos el Registry, el GameRegistry y al Supervisor dinámico en la aplicación.

```elixir
defmodule TheLiveCounter.Application do
  @moduledoc false

  use Application

  def start(_type, _args) do
    children = [
      # Another supervised processes
      {DynamicSupervisor, strategy: :one_for_one, name: TheLiveCounter.DynamicSupervisor},
      TheLiveCounter.GameRegistry,
      {Registry, keys: :unique, name: Registry.ViaGame}
    ]

    opts = [strategy: :one_for_one, name: TheLiveCounter.Supervisor]
    Supervisor.start_link(children, opts)
  end

  ## More code...

end
```

Con ello podríamos hacer llamados cómo los siguientes:

```elixir
{:ok, name, _} = TheLiveCounter.GameRegistry.create()
# {:ok, "qzufjr", #PID<0.322.0>}
TheLiveCounter.Game.get(name)
# %TheLiveCounter.Game{counter: 0, id: 386, name: "qzufjr"}
```

Y aunque no se manejan explícitamente los procesos, sabemos que están en acción por debajo de dichas llamadas.

Para finalizar, una de las cosas que ofrece `Registry` es el hecho de que si el proceso termina por cualquier razón, entonces también es eliminado del registro, lo cuál hace más sencillo su manejo.

[ets]: https://erlang.org/doc/man/ets.html
[registry]: https://hexdocs.pm/elixir/master/Registry.html
