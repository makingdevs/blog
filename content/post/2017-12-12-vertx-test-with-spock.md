+++
tags = []
draft = false
date = "2017-12-12T16:34:54-05:00"
title = "Testing Vert.x with Spock"
description = ""
author="neodevelop"
+++
Recently I was trying to test some components in Vert.x, to make sure of the behavior of the message's receptions, how to store correctly in the Shared Map?, is the value in the Map?, and so on...

So, Vert.x includes a [testing tool][1], is based in JUnit, the best, it has a Runner the `VertxUnitRunner.class` for **JUnit**, and for scripts tests you can use `TestSuite.create`, but what is the thing, the sentence: `Async async = context.async();`, and you should call the `complete()` method of the `async` object to end the test. Something like this:

{{<highlight groovy>}}
Async async2 = context.async();
vertx.eventBus().consumer("the-address", msg -> {
  async2.complete();
});
{{</highlight>}}

Well, I don't say this is bad, or better, for me is an option, a good option. But I want to go more natural, also we are using Groovy, but if you are a Java developer you should give a chance to the JUnit support.

This implementation maybe is not the best, we accept suggestions, but is the beginning of a set of Specs that we can define inside the project, and I want to show a simple setup, in some simple steps:

The important parts in **build.gradle** are:

{{<highlight groovy>}}
dependencies {
  compile 'org.slf4j:slf4j-api:1.7.21'
  compile "io.vertx:vertx-lang-groovy:3.5.0"  // Using Groovy !!!
  compile 'ch.qos.logback:logback-classic:1.2.3'
  testCompile "junit:junit:4.12"
  testCompile 'org.spockframework:spock-core:0.7-groovy-2.0' // Spock rulz !!!!
}

// This is useful for Spock and Vert.x,
// to identify the files where the Vert.x API is used
processResources {
  from 'src/main/groovy'
}
{{</highlight>}}

The example only testing a simple consumer in the app, something like this:

{{<highlight groovy>}}
def eb = vertx.eventBus()

eb.consumer("com.makingdevs.ping") { msg ->
  // Maybe we will use shared map in other post!!!
  eb.send("com.makingdevs.pong", "Pong!")
}
{{</highlight>}}

The consumer is waiting for a message, when receives it send another message, we want to test that behavior.

So, in the specification we use an instance of vertx, the directory where the verticles are located and a great component called [`PollingConditions`][2].

## The class [`PollingConditions`][2]

In brief, this class is useful to test async behaviours, those are executed in others threads not the main thread, so you have to wait for ending the executions inside them and compare wherever you want.

The key element is: `conditions.eventually { }`, I'm going to show it:

{{<highlight groovy>}}
package com.makingdevs.vertx

import io.vertx.core.Vertx
import spock.lang.*
import spock.util.concurrent.PollingConditions

@Stepwise
class PingPongVerticleSpec extends Specification {

  // We use the vertx instance provided
  @Shared Vertx vertx
  // Where are the verticle files?
  @Shared String baseDir = "com/makingdevs/vertx"
  // The verticle filename
  @Shared String verticleName = "ReceiverVerticle.groovy"
  // We use this id for deploy and undeploy
  @Shared String verticleId
  // Look ma! The async magic...!
  PollingConditions conditions = new PollingConditions()

  /*
  * For all the specs inside this file (executing once):
  * - Deploying the verticle with the vertx instance
  * - `assert` the deployment identifier
  * - Waiting for deployment
  */
  def setupSpec() {
    vertx = Vertx.vertx()
    vertx.deployVerticle("${baseDir}/${verticleName}") { deployResponse ->
      assert deployResponse.succeeded()
      verticleId = deployResponse.result
    }
    Thread.sleep 1000
  }

  /*
  * At the end of all the specs:
  * - Waiting for attend all the messages
  * - Undeploying the verticle
  * - Closing vertx resource
  */
  def cleanupSpec() {
    Thread.sleep 2000
    vertx.undeploy(verticleId) { response ->
      assert response.succeeded()
      vertx.close()
    }
  }

  def "Making Ping to a Verticle"(){
    given: "A message for send, and an empty response"
      def message = "Ping!"
      def response = ""
    and: "A consumer in vertx for waiting the response"
      vertx.eventBus().consumer("com.makingdevs.pong") { msg ->
        response = msg.body()
      }
    when: "We send a message, and wait..."
      vertx.eventBus().send("com.makingdevs.ping", message)
      conditions.eventually { // This should happen...
        assert response == "Pong!"
      }
    then: "We shouldn't thrown exception because conditions was satisfied..."
      noExceptionThrown()
  }

}
{{</highlight>}}

Here you are, tests the verticles declared in Vert.x with Spock, or alternatively called: Async Tests.

The entire code is in [GitHub ready to run the test][3], enjoy!

Be steady, and make tests to be happy.

[1]: http://vertx.io/docs/vertx-unit/java/
[2]: http://spockframework.org/spock/javadoc/1.0/spock/util/concurrent/PollingConditions.html
[3]: https://github.com/neodevelop/VertxTestsWithSpock