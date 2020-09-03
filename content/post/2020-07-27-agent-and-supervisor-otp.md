---
title: "Agentes y Supervisores en OTP"
author: "neodevelop"
type: "post"
date: 2020-07-26T17:09:15-05:00
subtitle: "¿Cómo construimos aplicaciones con OTP?"
tags: ["elixir","otp", "domino"]
---

Hace tiempo hemos estado construyendo un juego de [Domino](https://domino.makingdevs.com), basándonos sólo en el ecosistema proveído por Erlang/Elixir, y lo que deseo explicar aquí es una primer parte de los elementos que nos ayudaron a diseñarlo, sólo usando elementos basados en OTP, aquí explico el primero de ellos.

# Agentes en OTP

Cuándo explico el funcionamiento de los procesos y llegamos a la parte de recursividad, toco la posibilidad de ¿cómo un proceso puede retener estado?, y exactamente eso es un *Agente*, _abstracciones simples alrededor del estado_, los cuáles, compartírán y almacenarán estado que podrá ser accedido por otros procesos o por el mismo.

La [documentación de Elixir al respecto de Agentes](https://hexdocs.pm/elixir/Agent.html#content) provee de bastante información, de la cual tomaré lo esencial para detallar el uso que le dimos.

Por ejemplo, ocuparémos un contador, y aunque simple, muestra cómo al final todo se trata de transformar el estado, y escalar la forma de uso.

```elixir
{:ok, agent} = Agent.start_link fn -> 0 end # {:ok, #PID<0.327.0>}
Agent.update(agent, fn state -> state + 1 end) # :ok
Agent.get(agent, fn state -> state end) # 1
```

Y lo que queremos hacer es guardar el estado del juego, para lo cuál usare sólo un módulo, el cuál, contiene todos los elementos que queremos guardar que corresponden al juego, y algunas funciones que modifican la estructura.

```elixir
defmodule TheLiveCounter.Game do
  alias __MODULE__
  defstruct counter: 0, id: 0

  def create() do
    %Game{id: System.unique_integer([:positive])}
  end

  def increase(%Game{counter: counter} = game) do
    %Game{game | counter: counter + 1}
  end
end
```

# El supervisor dinámico

Lo que queremos es que este proceso sea creado, supervisado y en algún momento destruido, y que no sea el único proceso, para lo cuál, agregaremos un supervisor al árbol de supervisión dentro de nuestrop módulo principal en el archivo _application.ex_.

```elixir
def start(_type, _args) do
  children = [
    TheLiveCounterWeb.Telemetry,
    {Phoenix.PubSub, name: TheLiveCounter.PubSub},
    TheLiveCounterWeb.Endpoint,
    # NOTE: This is our supervisor
    {DynamicSupervisor, strategy: :one_for_one, name: TheLiveCounter.DynamicSupervisor}
  ]

  opts = [strategy: :one_for_one, name: TheLiveCounter.Supervisor]
  Supervisor.start_link(children, opts)
end
```

`TheLiveCounter.DynamicSupervisor` es el nombre por el cual referenciamos a nuestro proceso supervisor con el cuál podríamos obtener algún otro proceso que contenga a alguno de nuestros agentes.

# El supervisor dinámico y los agentes

Una vez hecho esto podemos ejecutar la `iex` con el contenido de la aplicación y comprobar que todo está en orden, ocupemos el siguiente script:

```elixir
alias TheLiveCounter.Game
{:ok, agent} = DynamicSupervisor.start_child(
  TheLiveCounter.DynamicSupervisor,
  {Agent, fn -> Game.create() end}) # {:ok, #PID<0.331.0>}
Agent.update(agent, fn game -> Game.increase(game) end) # :ok
Agent.get(agent, & &1) # %TheLiveCounter.Game{counter: 1, id: 2}
```

Enseguida, creamos varios agentes supervisados que contienen la estructura de `Game`, y podemos manejar cada uno de ellos:

```elixir
alias TheLiveCounter.Game
{:ok, another_agent} = DynamicSupervisor.start_child(
  TheLiveCounter.DynamicSupervisor,
  {Agent, fn -> Game.create() end}) # {:ok, #PID<0.334.0>}
Agent.get(another_agent, & &1) # %TheLiveCounter.Game{counter: 0, id: 34}
```

Es aquí en dónde podemos ocupar las funciones contenidas en el supervisor para obtener información al respecto de lo que está manejando, por ejemplo:

```elixir
DynamicSupervisor.count_children(TheLiveCounter.DynamicSupervisor)
# %{active: 2, specs: 2, supervisors: 0, workers: 2}
DynamicSupervisor.which_children(TheLiveCounter.DynamicSupervisor)
# [
#   {:undefined, #PID<0.331.0>, :worker, [Agent]},
#   {:undefined, #PID<0.344.0>, :worker, [Agent]}
# ]
```

## Esto es sólo el inicio

Con esto comenzamos a crear la estructura de almacenamiento y administración de los procesos que contienen el juego.

## Datos adicionales de este código

Para no agregar complejidad cree un proyecto muy simple, removiendo lo que no necesite para evitar confusiones:

`mix phx.new the_live_counter --live --no-ecto --no-gettext --no-dashboard`

El juego lo puedes encontrar en https://domino.makingdevs.com y estamos agregando algunas funciones extras para que lo disfrutes mejor.

### El repositorio

Lo podrás encontrar en [GitHub de MakingDevs](https://github.com/makingdevs/the_live_counter), en dónde lo estaré trabajando para estas publicaciones.
