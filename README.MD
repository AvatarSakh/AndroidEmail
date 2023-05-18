<h2 align="center">  МИНИСТЕРСТВО НАУКИ И ВЫСШЕГО ОБРАЗОВАНИЯ РОССИЙСКОЙ ФЕДЕРАЦИИ ФЕДЕРАЛЬНОЕ ГОСУДАРСТВЕННОЕ БЮДЖЕТНОЕ ОБРАЗОВАТЕЛЬНОЕ УЧРЕЖДЕНИЕ ВЫСШЕГО ОБРАЗОВАНИЯ «САХАЛИНСКИЙ ГОСУДАРСТВЕННЫЙ УНИВЕРСИТЕТ» </h2>
<div align="center">
<h3>Институт естественных наук и техносферной безопасности
<br>
Кафедра информатики
<br>
Половников Владислав Олегович</h3>

<br>
<h3>Дополнительное задание №2
<br>
“Email”
<br>
01.03.02 Прикладная математика и информатика</h3>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<h3 align="right">Научный руководитель
<br>
Соболев Евгений Игоревич
</h3>

<h3 align="center">Южно-Сахалинск
<br>
2023г.
</h3>
<hr>
</div>
<p>
Kotlin - это высокоэффективный современный язык программирования, разработанный компанией JetBrains. Он имеет очень чёткий и лаконичный синтаксис, что делает ваш код лёгким для чтения. Kotlin широко используется во всём мире, и его популярность среди разработчиков постоянно растёт. Многие разработчики, использующие Kotlin, отмечают, что это делает их работу быстрее и увлекательнее.
</p>

<h3 align="center">Задание</h3>

Проверка адреса email
Создать хороший UX для пользователей, вводящих адрес электронной почты и пароль при регистрации в приложении.

Требования:

Проверка формата электронной почты. Пример: user@gmail не является действительным адресом электронной почты
Пользовательский интерфейс должен показывать, действителен или нет адрес электронной почты. При необходимости интерфейс должен указать, что не так с адресом
Автозаполнение и проверка доступности домена. Пользователи часто опечатываются при вводе адреса. Например, указывают неправильно доменное имя (gmail.con вместо gmail.com)
Проверка пароля. Нет ограничения на вводимые символы. Есть ограничение минимальной и максимальной длины
При необходимости, интерфейс должен указать, что неправильно
Проверить, что заполнены все поля, и указать, какое именно не заполнено
Для автозаполнения необходимо:

Проверить существование введённого домена
Указать, что неправильно в введённом имени
Предложить Автозаполнение доменного имени самыми вероятными и популярными доменными именами. Пример: если пользователь вводит «user@», то продолжениями могут быть «user@gmail.com», «user@yahoo.com» и т.д. Если пользователь уточняет «user@g», то продолжениями могут быть популярные домены, начинающиеся с «g». Например: «user@gmail.com», «user@gmail.co.uk»





<h1 align="center">Решение</h2>

![img1](/images/1.png)

![img2](/images/2.png)

![img3](/images/3.png)

![img4](/images/4.png)

![img5](/images/5.png)

![img6](/images/6.png)

<h2 align="center">MainActivity</h2>

```kotlin
package com.example.lab7

import android.os.Bundle
import android.text.Editable
import android.text.TextUtils
import android.text.TextWatcher
import android.util.Patterns
import android.widget.*
import androidx.appcompat.app.AppCompatActivity
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.runBlocking
import kotlinx.coroutines.withContext
import java.net.InetAddress
import java.net.UnknownHostException
import java.util.concurrent.Executor
import java.util.concurrent.Executors


class MainActivity : AppCompatActivity(), PingCallback {

    private lateinit var emailText: AutoCompleteTextView
    private lateinit var passwordText: EditText
    private lateinit var loginButton: Button
    private lateinit var error_text1: TextView
    private lateinit var error_text2: TextView

    private var emailSuggestions = arrayOf("gmail.com", "gmail.uk.com", "yahoo.com", "hotmail.com", "mail.ru","bk.ru","yandex.ru")
    override fun onCreate(savedInstanceState: Bundle?)  {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)




        emailText = findViewById(R.id.editText_email)
        passwordText = findViewById(R.id.editText_password)
        loginButton = findViewById(R.id.button_login)
        error_text1 = findViewById(R.id.error_text1)
        error_text2 = findViewById(R.id.error_text2)

        //создаем подсказку
        var adapter = ArrayAdapter(this, android.R.layout.simple_dropdown_item_1line, emailSuggestions)
        emailText.setAdapter(adapter)

        emailText.addTextChangedListener(object : TextWatcher {
            override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {
            }

            override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {
                val input = s.toString()
                //подбор подсказки по введеному тексту
                if (input.contains("@")) {
                    val index = input.indexOf("@")
                    val domain = input.substring(index + 1)
                    val temparray = emailSuggestions.filter { it.startsWith(domain) }
                    val combinedEmails = mutableListOf<String>()
                    for (suggestion in temparray) {
                        val email = input.replaceAfter("@", "") + suggestion
                        combinedEmails.add(email)

                    }
                    val resultArray = combinedEmails.toTypedArray()
                    adapter = ArrayAdapter(this@MainActivity, android.R.layout.simple_dropdown_item_1line, resultArray)
                    emailText.setAdapter(adapter)
                }

            }

            override fun afterTextChanged(s: Editable?) {

            }

        })


        loginButton.setOnClickListener{
            val email = emailText.text.toString()
            val password = passwordText.text.toString()
            var textFail = false



            //проверка на пустое поле password
            if (TextUtils.isEmpty(password)) {
                error_text2.setText(R.string.empty)
                textFail = true
            }
            else{
                //ограничение пароля по вводим символам
                val minLengthChar = 3
                val maxLengthChar = 8
                error_text2.setText("")
                if(password.length<minLengthChar){
                    error_text2.setText(R.string.low_password)
                    textFail = true
                }
                if(password.length>maxLengthChar){
                    error_text2.setText(R.string.high_password)
                    textFail = true
                }
            }


            //проверка на пустое поле email
            if (TextUtils.isEmpty(email)) {
                error_text1.setText(R.string.empty)
                textFail = true
            }else {
                error_text1.setText("")
                //проверка на доступность домена
                val domain = email.substring(email.indexOf("@") + 1)
                if(textFail == false){
                    startPingAndContinueWork(domain)
                }
            }

        }


    }
    override fun onPingResult(success: Boolean) {
        runOnUiThread { if (success) {
            error_text1.setText("")
            val email = emailText.text.toString()
            val password = passwordText.text.toString()
            val foundAccount = accountList.find{ it.login == email}
            if(foundAccount == null) Toast.makeText(this,R.string.fail_login,Toast.LENGTH_SHORT).show()
            else {
                if(foundAccount.password != password) Toast.makeText(this,R.string.fail_login,Toast.LENGTH_SHORT).show()
                else Toast.makeText(this,R.string.success_login,Toast.LENGTH_SHORT).show()
            }
        } else {
            error_text1.setText("Domain isn't correct")
        }}
    }
    private val executor: Executor = Executors.newSingleThreadExecutor()
    fun startPingAndContinueWork(domain: String) {
        executor.execute(PingTask(domain, this))
    }
}

interface PingCallback {
    fun onPingResult(success: Boolean)
}

class PingTask(private val domain: String, private val callback: PingCallback) : Runnable {
    override fun run() {
        try {
            val address = InetAddress.getByName(domain)
            val success = address.isReachable(3000) 
            callback.onPingResult(success)
        } catch (e: UnknownHostException) {
            callback.onPingResult(false)
        }
    }
}

```

<h2 align="center">Accounts</h2>

```kotlin
package com.example.lab7


data class Accounts(val login: String, val password: String)

var accountList = listOf(
    Accounts("admin@mail.ru","1234"), 
    Accounts("test1@mail.ru","111"))

```


<h2 align="center">activity_main</h2>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">


    <TextView
        android:id="@+id/Top_background"
        android:layout_width="match_parent"
        android:layout_height="150dp"
        android:background="@color/orange"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/Top_textview"
        android:layout_width="0dp"
        android:layout_height="70dp"
        android:layout_marginStart="32dp"
        android:layout_marginTop="48dp"
        android:layout_weight="1"
        android:text="@string/top_text"
        android:textColor="@color/white"
        android:textSize="30dp"
        android:textStyle="bold"
        app:layout_constraintEnd_toStartOf="@+id/Top_image"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <ImageView
        android:id="@+id/Top_image"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_marginTop="32dp"
        android:layout_marginEnd="16dp"
        android:layout_weight="1"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:srcCompat="@drawable/quest" />

    <ImageView
        android:id="@+id/imageWaves"
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:layout_marginTop="-5dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/Top_background"
        app:srcCompat="@drawable/background" />

    <LinearLayout
        android:id="@+id/layout_email"
        android:layout_width="0dp"
        android:layout_height="60dp"
        android:layout_marginStart="32dp"
        android:layout_marginEnd="32dp"
        android:orientation="horizontal"
        android:background="@drawable/rounded_orange"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/imageWaves" >

        <ImageView
            android:id="@+id/imageView"
            android:layout_width="35dp"
            android:layout_height="35dp"
            android:layout_gravity="center"
            android:layout_marginLeft="20dp"
            app:srcCompat="@drawable/email" />

        <AutoCompleteTextView
            android:id="@+id/editText_email"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:background="@android:color/transparent"
            android:hint="@string/login_email"
            android:layout_marginLeft="10dp"
            android:layout_marginRight="15dp"
            android:layout_weight="1"
            android:inputType="textEmailAddress" />

    </LinearLayout>

    <TextView
        android:id="@+id/error_text1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text=""
        tools:text="email_error"
        android:textColor="@color/red"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/layout_email" />

    <LinearLayout
        android:id="@+id/layout_password"
        android:layout_width="0dp"
        android:layout_height="60dp"
        android:layout_marginStart="32dp"
        android:layout_marginEnd="32dp"
        android:background="@drawable/rounded_orange"
        android:orientation="horizontal"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/error_text1">

        <ImageView
            android:id="@+id/imageView2"
            android:layout_width="35dp"
            android:layout_height="38dp"
            android:layout_gravity="center"
            android:layout_marginLeft="20dp"
            app:srcCompat="@drawable/closed" />

        <EditText
            android:id="@+id/editText_password"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_marginLeft="10dp"
            android:layout_marginRight="15dp"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:ems="10"
            android:hint="@string/login_password"
            android:inputType="textEmailAddress" />
    </LinearLayout>

    <TextView
        android:id="@+id/error_text2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text=""
        tools:text="password_error"
        android:textColor="@color/red"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/layout_password" />

    <androidx.appcompat.widget.AppCompatButton
        android:id="@+id/button_login"
        android:layout_width="0dp"
        android:layout_height="60dp"
        android:layout_marginStart="32dp"
        android:layout_marginEnd="32dp"
        android:background="@drawable/rounded_orange_solid"
        android:text="@string/button_login"
        android:textColor="@color/white"
        android:textSize="20dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/error_text2"
        tools:ignore="MissingConstraints" />

</androidx.constraintlayout.widget.ConstraintLayout>

```

<h2 align="center">strings</h2>

```kotlin
<resources>
    <string name="app_name">lab7</string>
    <string name="login_email">Email</string>
    <string name="login_password">Password</string>
    <string name="button_login">Login</string>
    <string name="sign_with">OR</string>
    <string name="success_login">Success login</string>
    <string name="fail_login">Incorrect login or password</string>
    <string name="empty_login">Write your username</string>
    <string name="empty_password">Write your password</string>
    <string name="top_text">Already Have an Account?</string>
    <string name="error_change_password">Email isn\'t exist</string>
    <string name="success_change">Message with link for change password send on your email</string>
    <string name="incorrect_email">Incorrect email!</string>
    <string name="low_password">Few characters entered</string>
    <string name="high_password">Too many characters entered</string>
    <string name="empty">This input is empty</string>
</resources>
```
<h2 align="center">Вывод</h2>
Я научился создавать подсказки при вводе почты