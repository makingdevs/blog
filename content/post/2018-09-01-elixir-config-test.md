+++
tags = ["elixir","test","config"]
draft = true
date = "2018-09-04T19:22:54-05:00"
title = "Entornos y configuración en Elixir"
description = "Entornos y configuración en Elixir"
author="neodevelop"
+++

Al estar desarollando un par de proyectos en Elixir, me di a la labor de tanto meter pruebas como usar elementos que simulen llamadas a otros sistemas. Y descubrí que Elixir cuenta con ello de formas distintas a lo que ya había manejado en otras plataformas, y sólo quiero ejemplificar un par de ellos, se que puede haber más o mejores pero estos me sirvieron para ciertos própositos.

## Simulación de componentes

Mientras estaba desarrollando algo con Nerves y usando la cámara conectada a un Rasperry, me di cuenta que la biblioteca que manejaba la cámara contaba con una implementación que proveía la manera de simular el tomar una foto, dentro del archivo de configuración de la aplicacion `config.exs` podemos hacer algo como lo siguiente:

{{<highlight elixir>}}
use Mix.Config
config :picam, camera: Picam.FakeCamera
{{</highlight>}}

Y después en cualquier parte de nuestra aplicación donde lo necesitemos podemos usarlo, con ayuda del módulo `Application` :

{{<highlight elixir>}}
camera = Application.get_env(:picam, :camera, Picam.Camera)
children = [
  worker(camera, []),
  # ...
]
{{</highlight>}}

Explorando el código fuente en [el repositorio de Github](1), me percaté que lo único que hace es sustituir la implementación de las funciones que eventualmente tienen ambos, sólo así, sin ningún contrato de por medio, no podría decir que parecido a una interfaz o un protocolo, por que pues sólo es eso, la misma función en dos módulos distintos, una que sólo hace un simple log y otra que verdaderamente hace el trabajo.

Con ello pude hacer simular el comportamiento de varios componentes para la aplicación en un ambiente específico, comenzando por la llamada desde el archivo `config.exs`:

{{<highlight elixir>}}
use Mix.Config
import_config "#{Mix.env}.exs" # Esto llama al ambiente apropiado
{{</highlight>}}

Y el archivo de configuración de desarrollo y test, `dev.exs` y `test.exs`, me quedan como:

{{<highlight elixir>}}
use Mix.Config

config :picam, camera: Picam.FakeCamera # Módulo incluido en la biblioteca
config :raspi3, uart: Raspi3.UART.Fake # Módulo creado por mí
{{</highlight>}}

Y el archivo de configuración de producción queda como:

{{<highlight elixir>}}
use Mix.Config

# Look ma! No puse el módulo de la cámara, ¿por qué?
config :raspi3, uart: Nerves.UART
{{</highlight>}}

Y al usarlos dentro de la aplicación los pude llamar de la siguiente forma:

{{<highlight elixir>}}
camera = Application.get_env(:picam, :camera, Picam.Camera)
uart = Application.get_env(:raspi3, :uart)

children = [
  {uart, [name: Raspi3.Arduino.Serial]},
  camera
]
{{</highlight>}}

Lo pongo así con fines demostrativos, por que la función `Application.get_env/3` ayuda a obtener el valor desde la configuración, pero si no lo encuentra pone por defecto el que está como tercer argumento de la función, es por eso que no es necesario que en la configuración de producción lo especifíque.

## Pruebas dentro de la aplicación

La situación ahora es comprobar que el comportamiento de mi aplicación, especialmente un *GenServer*, en donde tengo un colaborador el cual quiero saber que es invocado. El módulo de prueba es algo cómo:

{{<highlight elixir>}}
defmodule App.Manager do

  @timeout 60000

  def send_command(command_id, collaborator_action \\ RealModuleOperation) when is_number(command_id) do
    :poolboy.transaction(
      App.SomeServer.Pool,
      fn pid ->
        GenServer.cast(pid, {:send_command, command_id, collaborator_action}) # Módulo colaborador
      end,
      @timeout
    )
  end

end
{{</highlight>}}

Lo que me gustó aquí fue la posibilidad de enviar el segundo parámetro de la función `send_command` con un valor por defecto, el cuál, es el módulo que tiene la lógica real que quiero ejecutar y me ayudará a enviarle cualquier otro código que quiera ejecutar cómo módulo. Por ejemplo en una prueba podría hacer algo cómo:

{{<highlight elixir>}}
defmodule App.ManagerTest do
  use App.RepoCase

  alias App.Manager

  test "make an invocation" do

    ## NOTE: Register for scope inside the module
    Process.register self(), :manager_test
    defmodule OperationFake do
      def collaboration_call(_some_model) do
        send :manager_test, :send_confirmation
      end
    end

    Manager.send_command(1, OperationFake)

    assert_received :send_confirmation
  end
end
{{</highlight>}}

Puedo enviarle un módulo declarado en línea, el cuál, registra el proceso actual para poder enviar un mensaje, y gracias a `assert_received` asegurar a que efectivamente ese elemento fue invocado.

Esto lo tomé en base al artículo de José Valim de nombre: [Mocks and explicit contracts][2]

Seguramente hay más formas de hacer simulación de componentes y/o ejecutar elementos por ambiente, sin embargo estas son las formas que de primera mano me han servido.

[1]: https://github.com/electricshaman/picam
[2]: http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/