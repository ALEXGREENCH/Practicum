

## Задание 1

**Привет!** 👋  
Использование `OnSharedPreferenceChangeListener` – это отличное решение для отслеживания изменений в настройках.

Но есть пара нюансов, которые можно улучшить, чтобы код стал надёжнее и удобнее.

**Проблема:**
Сейчас `listener` создаётся внутри `setPreferencesListener`, но нигде не сохраняется. Это может привести к тому, что `SharedPreferences` будет продолжать удерживать ссылку на `listener`, даже после выхода из экрана, что вызовет утечку памяти.


**Как исправить:**
Оптимальное решение — объявить `listener` как переменную класса и управлять его регистрацией вручную:

```kotlin
private  var listener: SharedPreferences.OnSharedPreferenceChangeListener?  =  null

fun  startListening()  {
	listener = SharedPreferences.OnSharedPreferenceChangeListener  { _, key ->
		if  (key == HISTORY_LIST_KEY)  updateHistory()  
	}  
	sharedPrefs.registerOnSharedPreferenceChangeListener(listener)  
}  

fun  stopListening() {  
		listener?.let { sharedPrefs.unregisterOnSharedPreferenceChangeListener(it) }  
}
```

----------

## Задание 2

Давай разберёмся, как сделать код лаконичнее и надежнее.

###  Улучшение 1: `when` вместо `if-else` (в качестве примера посмотреть, не итоговое)

Цепочку `if-else` можно заменить `when`, чтобы сделать код чище:

```kotlin
fun turnTo(direction: String) {
    when (direction) {
        "North" -> northAction()
        "East" -> eastAction()
        "South" -> southAction()
        "West" -> westAction()
        else -> println("Invalid direction")
    }
}

```

📌 `when` удобнее и понятнее if-else конструкций, но конкретно для направлений не совсем подходит, о чём ниже.

### Улучшение 2: Используем `enum`

Чтобы избежать ошибок из-за опечаток в строках, можно заменить `String` на `enum`:

```kotlin
enum class Direction { NORTH, EAST, SOUTH, WEST }

fun turnTo(direction: Direction) {
    when (direction) {
        Direction.NORTH -> northAction()
        Direction.EAST -> eastAction()
        Direction.SOUTH -> southAction()
        Direction.WEST -> westAction()
    }
}

```

✅ `enum` делает код безопаснее, компилятор следит за корректностью значений.

**Что почитать:**  
[Kotlin Enum Classes](https://kotlinlang.org/docs/enum-classes.html)

----------
## Задание 3
  
Твой код уже работает, но есть способы сделать его более эффективным и читаемым.

**Проблема:**
Конкатенация строк в цикле через  `+`  создаёт много временных объектов.

**Решение:**
Замени на  `joinToString()`:
```kotlin
fun createResultString(users: List<User>): String = 
	users.joinToString("\n") { "Имя: ${it.name}, Возраст: ${it.age}" }
```


**Совет:**  
Для больших данных (10k+ элементов) используй `StringBuilder`, чтобы избежать падения производительности.

**Пример:**
```kotlin
val sb = StringBuilder() users.forEach { sb.append("${it.name}\n") } 
return sb.toString()
```

Теперь код компактнее и понятнее! 🚀

**Что почитать:**  
[Concatenate Strings in Kotlin](https://www.baeldung.com/kotlin/concatenate-strings)

----------

## Задание 4

**Есть пару замечаний:**
1.  Ты забыл вызвать  `apply()`  или  `commit()`, поэтому изменения не сохраняются.
2.  Опечатка в ключе для  `promoWasShown`  (используется  `ONBOARDING_KEY`).

Добавляем `.apply()`, чтобы изменения сохранялись:

```kotlin
var promoWasShown: Boolean
	set(value) = preferences.edit { putBoolean(PROMO_KEY, value) }.apply()

```
Аналогично исправляем `promoWasShown`.

Все ключи должны быть уникальными и соответствовать свойствам.

**Что почитать:**  
[SharedPreferences](https://developer.android.com/reference/android/content/SharedPreferences.html)
[Android Security: EncryptedSharedPreferences](https://developer.android.com/reference/androidx/security/crypto/EncryptedSharedPreferences)

----------

## Задание 5

**Есть пару замечаний:**
1.  `textView`  инициализируется до  `setContentView()`, что приводит к NPE.
2.  Таймер не отменяется в  `onDestroy()`, вызывая утечку памяти.
3.  В качестве ссылки на `Context` для `Toast` указан `this@TimerActivityIn`, но такого класса нет. 

✅ **Доработка кода:**

```kotlin
private var textView: TextView? = null // можно как latinit, но не советую

override fun onCreate(savedInstanceState: Bundle?) { 
	super.onCreate(savedInstanceState)
	setContentView(R.layout.activity_timer) // сначала разметка!
	textView = findViewById(R.id.text)
	countDownTimer = object : CountDownTimer(TIME, INTERVAL){
	
		override fun onTick(millisUntilFinished: Long) {
	                textView?.text = getString(R.string.millis_until_finished, millisUntilFinished.toString())
		}

		override fun onFinish() {
		    Toast.makeText(this@TimerActivity, R.string.timer_is_finished, Toast.LENGTH_SHORT).show()
		}
	}.apply { start() } 
}

override fun onDestroy() {
	countDownTimer?.cancel() // останови таймер!
	super.onDestroy()
}

```
----------

## Задание 6

**Проблемы:**

1.  Использование  `!!`  для  `locationClient`  может вызвать NPE.
2.  Нет обработки отзыва разрешений.

✅ **Доработка кода:**

```kotlin
locationClient?.requestLocationUpdates(...)  // безопасный вызов

// Проверяй разрешения в `onResume()`:
if  (hasLocationPermission())  {  
	startLocationUpdates()  
}  else  {  
	requestPermissions()  
}
```
Так же логику можно вынести в отдельный класс, например,  `LocationProvider`.

**Что почитать:**  
[Requesting Location Permissions](https://developer.android.com/develop/sensors-and-location/location/permissions)

----------

## Задание 7

**Замечание:**
Не вызывается  `notifyDataSetChanged()`  после изменения данных, соответственно и визуально список не изменяется. 

✅ **Как исправить:**

```kotlin
fun addItem(user: User) {
	items.add(user)
	notifyItemInserted(items.size - 1) // уведоми адаптер! 
}
```

**Оптимизация:**  
Используй `DiffUtil` для эффективного обновления:
```kotlin
val diffResult = DiffUtil.calculateDiff(UserDiffCallback(oldItems, newItems)) 
items.clear()
items.addAll(newItems)
diffResult.dispatchUpdatesTo(this)
```

**Что почитать:**  
[Using DiffUtil in RecyclerView](https://medium.com/@iammert/using-diffutil-in-android-recyclerview-bdca8e4fbb00)

----------

Если есть вопросы или что-то непонятно — пиши, разберёмся! 😊
