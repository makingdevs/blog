+++
tags = ["elixir","nerves","phoenix","phoenixframework"]
draft = false
date = "2018-06-17T16:34:54-05:00"
title = "¿Cómo adopté Elixir?"
description = "¿Cómo comencé a usar Elixir?"
author="neodevelop"
+++

LLevo ya un par de años escuchando y haciendo Elixir, no con la intensidad que quisiera pero tampoco lo he perdido de vista; soy un programador con preferencias dinámicas opcionales, sin embargo, me gusta mucho la _magia_(compilador) que *Haskell* ofrece al programar.

De hace ya un par de años he querido comenzar una serie de artículos basados en LISP, sin embargo, el trabajo y los compromisos empresariales no lo han permitido, creo importante que los programadores conozcan las bases de la programación funcional, no por ser mejor, si no por que permite mejorar el/los paradigmas que estés usando. Al no tener la oportunidad de hacerlo, y ver que Elixir tiene/cuenta/cubre con los conceptos de la programación funcional y además pueden crearse sistemas productivos interesantes(no es que con Haskell o LISP no se pueda, más bien es una cosa de contextos), pues la he tomado como una buena opción para comenzar para mçi y el equipo con el que trabajo.

El punto en este texto no es platicar la introducción a Elixir, de eso hay ya varias publicaciones, más bien redactar como se me ha presentado de a poco la oportunidad de subirme a una plataforma como Erlang y Elixr, y con ello escribir una muy breve serie de artículos que muestren como resolví problemas, o use bibliotecas de tal manera que no encontré una explicación que me ayudará lo suficiente, o poner esos pequeños temas que creo que son importantes en el diseño de una app con Elixir.

## [Barcamp de diseño de software](https://www.youtube.com/watch?v=SWwEMYMDo_o)

### Artesanos de software 2013

En esta charla, @hiphoox nos platica de como es que está diseñado el lenguaje Elixir y la forma en que los sistemas pueden crecer, parafraseando comenta que en su opinión se le hace la forma adecuada en la cual un sistema tiene que crecer.

Este evento ha sido sin duda de los más reveladores e interesantes, no se ha repetido desde entonces algo similar.

En este momento el lenguaje elixir estaba en su version v0.12.0.

## Workshop de Elixir por Hiphoox

Poco tiempo después, el mismo @hiphoox organizó un pequeño taller de 4 días con donde mostó el poder del lenguaje y de un framweork llamado Phoenix en sus primeras versions.

De aquí, por más que busqué no encontré fotos o código que me ayudarán a recordar más al respecto, sin embargo, las sesiones que tomé me ayudaron a comprender el poder del lenguaje, del framework, pero sobre todo de algo denominado OTP.

## [Elixir Friends 2015](https://github.com/ElixirFriendsMx/elixir_dojo_mx)

Una serie de sesiones realizadas y organizadas por @MachinesAreUs, en donde pude adentrarme mucho más al lenguaje. La intención: Llegar mejor preparado al evento que se iba a realizar próximamente. 

Las sesiones muy intensas, pues eran vespertinas y nocturnas, la cooperación y el ánimo de todos me ayudaron a resistir después de jornadas de trabajo, en retrospectiva, fue un trabajo inspirador hecho por Agustín. El lenguaje me hacía más sentido pero empecé a tener las dudas más grandes al respecto del poder de la plataforma: Los procesos.

## [Erlang Factory Lite 2015](http://www.erlang-factory.com/mexico2015)

El primer evento de Erlang en México, en donde, José Valim creador de Elixir, vino a platicarnos del diseño del framwork Phoenix, mayormente sentí la emoción de conocer muy poco de lo que se hablaba y el interés que se generó alrededor. 

El ver como crearon las abstracciones que permitían usar el framework, y sin saberlo usar el poder de los procesos por debajo fue sorprendente; crear sistemas de ese estilo es lo que se necesita, es lo que tenía que empezar a hacer.

## [Code Camp Dic 2016 - Twinder](https://github.com/CodeCampMx/twinder/tree/master/lib)

https://www.meetup.com/es-ES/meetup-group-JfQVZcAL/photos/27466559/456582599/?photoId=456582599&photoAlbumId=27466559

Un evento organizado por Artesanos de Software, en donde mentores y aprendices se juntan para transmitir conocimiento, una prueba que me ayudo a mostrar como mentor mi habilidad en el lenguaje, con un par de personas que estaban interesadas y que era su primer acercamiento, y como aprendizaje validado, tenía que aterrizar en claro todas las ideas y conceptos de tal manera que fueran condensados, me di cuenta, que sabía de la sintaxis, pero me faltaba mucho por aprender al respecto de la plataforma, es decir, procesos y OTP.

## [Erlang Factory Lite 2016](http://www.erlang-factory.com/mexico2016)

El siguiente evento de Erlang en México, realizado en WeWork, incluso recuerdo que fui a apoyar a @hiphoox a guíar turísticamente a Bruce Tate con las cervezas, realmente todo un honor.

Era muy claro que Erlang es la plataforma a adoptar, y Elixir el lenguaje, sin embargo, me faltaba ponerlo en las prioridades tanto personales como empresariales, el evento estuvo lleno de temas que tenía que profundizar, la decisión en su momento fue adoptarlo pero de forma empresarial, es decir, proveer una línea de desarrollo basada en Elixir.

## [Entrenamiento en MD 2017](https://github.com/makingdevs/elixir-training-0217)

Eventualmente, organizamos con ayuda de @hiphoox un entrenamiento en MakingDevs, que permitierá acercar a la gente al conocimiento formal del lenguaje, de la plataforma y de Phoenix; fueron varias sesiones que nos ayudaron a comprender, la sintaxis, OTP y como construir aplicaciones distribuidas y escalables. En breve, fue revelador.

## [Elixir Conf 2017](http://elixirconf.mx/)

Este evento tuvo una charlas inspiradoras que alinearon lo que alguna vez quise hacer mientras estudiaba, mi meta era desarrollar hardware y software a medida, y aunque hoy es el día en que totalmente no desarrollo hardware, me emociona mucho saber lo que se puede hacer, he visto en el sitio del proyecto Nerves varios ejemplos que son verdaderamente sorprendentes, y la primer charla fue dada por Justin Schneck el creador de Nerves, donde dió una sesión introductoria y demostrativa del poder de Elixir en un hardware muy pequeño como el Raspberry. He de confesar que terminando la charla de Justin hice el pedido de un Raspberry Pi 3 en Amazon.

Otro suceso que me agradó fue ver a un ex-integrante del equipo dando una charla en el mismo evento, pues fue la apuesta que hicimos para crecer, y observar el nivel de ponencia que dió fue por mucho gratificante, saber que la gente puede tomar sus propias decisiones y actuar en consecuencia en lo que a nuevas tecnologías se refiere es parte del objetivo que tenemos.

## LunaFreya

Una vez con todo esto encima, me decidí a retomar "el camino del alquimista" y tocar Elixir pero subiéndome al Raspberry que recientemente compré, empecé capturando datos físicos de un Arduino y transformándolos dentro del dispositivo para activar una cámara de múltiples formas. 

De hecho, de eso es lo que pienso escribir en diversas breves partes(no como esta) siguiendo a esta publicación,

## Material de estudio que me ha servido

Entre los recursos que me han servido(aparte de las documentaciones de los sitios) para el mejor entendimiento son las siguientes (aunque de algunos no los he cubierto al 100%):

- Elixir in action, Manning.
- Programming Elixir, Pragmatic Programmers.
- The Little Elixir & OTP Guidebook, Manning.
- Programming Phoenix, Pragmatic Programmers.
- Functional Web Development with Elixir, OTP, and Phoenix, Pragmatic Programmers.
- LearnElixir.tv
- LearnPhoenix.tv
- [Elixir School](https://elixirschool.com/es/)
- El meetup de Elixir en México
- [Learn you some Erlang for great good](http://learnyousomeerlang.com/content)
- [Erlang Programming](http://shop.oreilly.com/product/9780596518189.do)

Y aunque existen más lugares dónde se puede encontrar contenido de calidad, estos son los que para mi forma de aprendizaje me han servido.

## ¿Qué sigue?

Continuar el camino, crear cada vez más aplicaciones, afortunadamente algunos clientes se han convencido de usarlo(o los hemos convencido), y hemos de asumir algunos riesgos tanto para ellos como para nosotros, sin embargo, la emoción de cumplir la meta podrá ser suficiente para que lleguemos a cumplir la meta.

En este punto me doy cuenta que vengo siguiendo esto de un par de años, y que no cuento con el conocimiento que quisiera del lenguaje y la plataforma, sin embargo, eso siempre será motivante para profundizar cada vez más...