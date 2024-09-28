# Домашнее задание к занятию «10. Coroutines: Scopes, Cancellation, Supervision»

## Вопросы: Cancellation

### Вопрос №1

Отработает ли в этом коде строка `<--`? Поясните, почему да или нет.

```kotlin
fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
    }
    delay(100)
    job.cancelAndJoin()
}
```
Ответ:

Нет, строка println("ok") не отработает.
Причина: вызов job.cancelAndJoin() отменяет корутину job и все её дочерние корутины, которые еще не начали выполнение или находятся в состоянии ожидания. Поскольку корутина с println("ok") еще не начала выполнение (она задержана на 500 мс), она будет отменена вместе с job, и код не дойдет до строки println("ok").

### Вопрос №2

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        val child = launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
        delay(100)
        child.cancel()
    }
    delay(100)
    job.join()
}
```
Ответ:

Нет, строка println("ok") не отработает.
Причина: корутина child будет отменена до того, как дойдет до строки println("ok"), так как на 100-й мс выполнения будет вызвано child.cancel(), что приведет к её отмене.


## Вопросы: Exception Handling

### Вопрос №1

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    with(CoroutineScope(EmptyCoroutineContext)) {
        try {
            launch {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```

Ответ:

Нет, строка e.printStackTrace() не отработает.
Причина: исключения в корутинах, запущенных через launch, не перехватываются блоком try-catch, который находится снаружи корутины. Чтобы перехватить исключение, нужно использовать supervisorScope или CoroutineExceptionHandler.

### Вопрос №2

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
Ответ:

Да, строка e.printStackTrace() отработает.
Причина: исключение, выброшенное внутри coroutineScope, можно поймать с помощью блока try-catch, так как coroutineScope сам по себе не является корутиной, а скорее suspending-функцией, и исключения, выброшенные внутри неё, всплывают наружу.

### Вопрос №3

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
Ответ:

Да, строка e.printStackTrace() отработает.
Причина: как и в случае с coroutineScope, исключение, выброшенное внутри supervisorScope, можно перехватить с помощью блока try-catch.

### Вопрос №4

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
    Thread.sleep(1000)
}
```
Ответ:

Нет, строка println("ok") не отработает.
Причина: первое исключение, выброшенное в корутине launch, прервет выполнение всех дочерних корутин внутри coroutineScope, включая ту корутину, которая должна выбросить исключение на строке // <--.

### Вопрос №5

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```

Ответ:

Да, строка e.printStackTrace() отработает, но с нюансами.
Причина: исключение из второй корутины будет обработано блоком try-catch, но оно не повлияет на выполнение первой корутины внутри supervisorScope. Таким образом, первая корутина достигнет строки // <-- и выбросит исключение, которое снова будет перехвачено блоком try-catch.

### Вопрос №6

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
```

Ответ:

Нет, строка println("ok") не отработает.
Причина: выброшенное исключение в первой корутине приведет к отмене всей иерархии корутин, и код в корутине, который находится после задержки в 1000 мс, не будет выполнен.

### Вопрос №7

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext + SupervisorJob()).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
```
Ответ:

Да, строка println("ok") отработает.
Причина: использование SupervisorJob позволяет изолировать исключения между корутинами, поэтому первая корутина, задержанная на 1000 мс, выполнится независимо от того, что произошло в другой корутине.
