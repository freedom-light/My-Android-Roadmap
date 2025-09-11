# MainActivity
```kotlin
package com.example.a02_week_demo1

import android.os.Bundle
import android.widget.ImageView
import android.widget.TextView
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat

class MainActivity : AppCompatActivity() {
    lateinit var avatarResId: ImageView
    lateinit var nameText: TextView
    lateinit var introText: TextView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContentView(R.layout.activity_main)
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }
        val person = Person("enen", "happyhappyhappy")

        avatarResId = findViewById(R.id.avatarResId)
        nameText = findViewById(R.id.nameText)
        introText = findViewById(R.id.introText)

        setPersonData(person)
    }
    fun setPersonData(person: Person){
        nameText.text = person.name
        introText.text = person.introduce
    }
}
```
# Person(data class)
```kotlin
package com.example.a02_week_demo1

data class Person(
    val name: String,
    val introduce: String,
    val avatarResId: Int = R.drawable.ic_launcher_background
)
```

# activity_main.xml
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ImageView
        android:id="@+id/avatarResId"
        android:layout_width="153dp"
        android:layout_height="141dp"
        android:layout_marginStart="20dp"
        android:layout_marginTop="20dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:srcCompat="@drawable/ic_launcher_background"
        tools:srcCompat="@tools:sample/avatars" />

    <TextView
        android:id="@+id/nameText"
        android:layout_width="78dp"
        android:layout_height="25dp"
        android:layout_marginStart="48dp"
        android:layout_marginTop="20dp"
        android:text="@string/hint_name"
        app:layout_constraintStart_toEndOf="@+id/avatarResId"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/introText"
        android:layout_width="409dp"
        android:layout_height="316dp"
        android:layout_marginBottom="200dp"
        android:text="@string/hint_introduce"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

# strings.xml
```kotlin
<resources>
    <string name="app_name">02-Week-Demo1</string>
    <string name="hint_name">姓名</string>
    <string name="hint_introduce">简介</string>
</resources>
```
