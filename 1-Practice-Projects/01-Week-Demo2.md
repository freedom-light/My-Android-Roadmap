# MainActivity
```kotlin
package com.example.oneweektodolisttmp

import android.os.Bundle
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import android.widget.Button
import android.widget.EditText
import android.widget.ListView
import android.widget.ArrayAdapter
import android.widget.Toast

class MainActivity : AppCompatActivity() {
    val textList = mutableListOf<String>()
    lateinit var addButton : Button
    lateinit var taskInput: EditText
    lateinit var viewList: ListView
    lateinit var adapter: ArrayAdapter<String>

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContentView(R.layout.activity_main)

        // 确保你的应用内容不会被系统的 UI（如状态栏、导航栏）遮挡
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }

        addButton = findViewById(R.id.AddButton)
        taskInput = findViewById(R.id.taskInput)
        viewList = findViewById(R.id.viewList)

        // 创建，绑定适配器
        adapter = ArrayAdapter(this, android.R.layout.simple_list_item_1, textList)
        viewList.adapter = adapter

        // 绑定点击事件
        addButton.setOnClickListener{
            addTask()
        }
        viewList.setOnItemClickListener{_, _, position, _->
            removeView(position)
            true
        }
    }

    // 添加任务
    fun addTask(){
        val task = taskInput.text.toString()
        if(task.isNotEmpty()){
            textList.add(task)
            adapter.notifyDataSetChanged()
            taskInput.text.clear()
        }
    }
    // 单击删除任务
    fun removeView(position: Int){
        textList.removeAt(position)
        adapter.notifyDataSetChanged()
        Toast.makeText(this, "删除成功", Toast.LENGTH_SHORT).show()
    }

}
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

    <ListView
        android:id = "@+id/viewList"
        android:layout_width="409dp"
        android:layout_height="529dp"
        android:layout_marginTop="200dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <EditText
        android:id="@+id/taskInput"
        android:layout_width="247dp"
        android:layout_height="57dp"
        android:layout_marginStart="20dp"
        android:layout_marginTop="100dp"
        android:ems="10"
        android:inputType="text"
        android:text="@string/hint_enter_new_task"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/AddButton"
        android:layout_width="99dp"
        android:layout_height="55dp"
        android:layout_marginTop="100dp"
        android:layout_marginEnd="30dp"
        android:text="@string/button_add_task"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

# strings.xml
```kotlin
<resources>
    <string name="app_name">oneWeekTodoListTmp</string>
    <string name="hint_enter_new_task">请输入代添加任务</string>
    <string name="button_add_task">添加</string>
</resources>
```
