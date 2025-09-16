# MainActivity
```kotlin
package com.example.a03_week_demo2

import android.content.Intent
import android.net.Uri
import android.os.Bundle
import android.util.Log
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import android.widget.TextView
import android.widget.Toast
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import androidx.lifecycle.lifecycleScope
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.bumptech.glide.Glide
import com.example.a03_week_demo2.com.example.a03_week_demo2.data.NewsItem
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    companion object {
        private const val TAG = "MainActivity"
    }

    private var dataList: List<NewsItem> = emptyList()
    private lateinit var recyclerView: RecyclerView

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

        recyclerView = findViewById(R.id.recyclerView)

        // 启动网络请求，加载UI数据
        loadNewsData()
    }
    // 发起网络请求，获取新闻数据
    fun loadNewsData(){
        lifecycleScope.launch {
            val response: ApiResponse = RetrofitClient.newsApiService.getNewsList()
            // 处理API响应，code == 200 代表成功
            if(response.code == 200){
                dataList = response.result.newsList

                // 打印日志输出信息
                Log.d(TAG, "请求成功！共获取 ${dataList.size} 条新闻")
                dataList.forEachIndexed { index, item ->
                    Log.d(TAG, "NewsItem-$index 标题：${item.title} | 来源：${item.source} | 图片：${item.picUrl}")
                }
                // 用 RecyclerView 展示列表
                initRecyclerView(dataList)
            }
        }
    }

    fun initRecyclerView(dataList: List<NewsItem>){
        // 初始化列表布局
        recyclerView.layoutManager = LinearLayoutManager(this)

        // 绑定适配器
        val adapter = MyAdapter(dataList)
        recyclerView.adapter = adapter
    }
}

// ViewHolder类
class MyHolder(view: View): RecyclerView.ViewHolder(view){
    val image: ImageView = itemView.findViewById(R.id.imageView)
    val title: TextView = itemView.findViewById(R.id.hinttitle)
    val source: TextView = itemView.findViewById(R.id.hintsource)
    val ctime: TextView = itemView.findViewById(R.id.hintCtime)
}

// Adapter类
class MyAdapter(private val dataList: List<NewsItem>): RecyclerView.Adapter<MyHolder>(){
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyHolder {
        val itemView = LayoutInflater.from(parent.context).inflate(R.layout.newsitem, parent, false)
        return MyHolder(itemView)
    }

    override fun onBindViewHolder(holder: MyHolder, position: Int) {
        val currentItem = dataList[position]
        if(currentItem.picUrl.isNullOrEmpty()){
            Glide.with(holder.itemView.context)
                .load(R.drawable.ic_launcher_background)
                .into(holder.image)
        }else{
            Glide.with(holder.itemView.context)
                .load(currentItem.picUrl)
                .placeholder(R.drawable.ic_launcher_background) // 加载中显示的图片
                .error(R.drawable.ic_launcher_background)        // 加载失败显示的图片
                .into(holder.image)
        }
        holder.title.text = currentItem.title
        holder.source.text = currentItem.source
        holder.ctime.text = currentItem.ctime
        // 为条目添加点击事件，打开对应的新闻链接
        holder.image.setOnClickListener {
            if (currentItem.url.isNullOrEmpty()){
                Toast.makeText(holder.itemView.context, "链接无效", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }
            val intent = Intent(Intent.ACTION_VIEW, Uri.parse(currentItem.url))
            holder.itemView.context.startActivity(intent)
        }
        //Log.d("NewItem-${dataList[position].id}","url:${dataList[position].url} | picUrl:${dataList[position].picUrl}")
    }

    override fun getItemCount(): Int {
        return dataList.size
    }
}

```
# AndroidManifest.xml
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" >

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme._03WeekDemo2" >
        <activity
            android:name=".MainActivity"
            android:exported="true" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />


            </intent-filter>
        </activity>
    </application>

</manifest>
```

# ApiResponse
```kotlin
package com.example.a03_week_demo2

import com.example.a03_week_demo2.com.example.a03_week_demo2.data.NewsResult

data class ApiResponse(
    val code: Int,
    val msg: String,
    val result: NewsResult
)
```

# NewsResult
```kotlin
package com.example.a03_week_demo2.com.example.a03_week_demo2.data

import com.google.gson.annotations.SerializedName

data class NewsResult(
    @SerializedName("curpage")
    val curPage: Int,
    @SerializedName("allnum")
    val allNum: Int,
    @SerializedName("newslist")
    val newsList: List<NewsItem>
)
```

# NewsItem
```kotlin
package com.example.a03_week_demo2.com.example.a03_week_demo2.data

data class NewsItem(
    val id: String? = "",
    val ctime: String? = "",
    val title: String? = "",
    val description: String? = "",
    val source: String? = "",
    val picUrl: String? = "",
    val url: String? = ""
)
```

# NewsApiService
```kotlin
package com.example.a03_week_demo2

import retrofit2.http.GET
import retrofit2.http.Query

interface NewsApiService {
    @GET("api/tx/guonei")
    suspend fun getNewsList(
        @Query("key") key: String = "d8c6d4c75ba0",
        @Query("num") num: Int = 10
    ): ApiResponse
}
```

# RetrofitClient
```kotlin
package com.example.a03_week_demo2

import okhttp3.OkHttpClient
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitClient{
    val BASE_URL = "https://whyta.cn/"

    // 创建OkHttpClient
    // 使用了建造者模式，由许多可选配置，这里实现最简部分
    val okHttpClient = OkHttpClient.Builder().build()

    // 创建Retrofit实例
    val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .client(okHttpClient)
        .addConverterFactory(GsonConverterFactory.create())
        .build()


    val newsApiService: NewsApiService by lazy{
        retrofit.create(NewsApiService::class.java)
    }
}
```

# build.gradle.kts Module
```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
}

android {
    namespace = "com.example.a03_week_demo2"
    compileSdk = 36

    defaultConfig {
        applicationId = "com.example.a03_week_demo2"
        minSdk = 24
        targetSdk = 36
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }
    kotlinOptions {
        jvmTarget = "11"
    }
}

dependencies {
    implementation(libs.kotlinx.coroutines.android)

    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.appcompat)
    implementation(libs.material)
    implementation(libs.androidx.activity)
    implementation(libs.androidx.constraintlayout)
    implementation(libs.androidx.retrofit)
    implementation(libs.androidx.converter.gson)
    implementation(libs.androidx.gson)
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)

}
```

# libs.versions.toml
```kotlin
[versions]
agp = "8.12.2"
kotlin = "2.0.21"
coreKtx = "1.17.0"
junit = "4.13.2"
junitVersion = "1.3.0"
espressoCore = "3.7.0"
appcompat = "1.7.1"
material = "1.13.0"
activity = "1.10.1"
constraintlayout = "2.2.1"

retrofit = "2.9.0"
gson = "2.10.1"

coroutinesAndroid = "1.7.3"

[libraries]
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "coreKtx" }
junit = { group = "junit", name = "junit", version.ref = "junit" }
androidx-junit = { group = "androidx.test.ext", name = "junit", version.ref = "junitVersion" }
androidx-espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "espressoCore" }
androidx-appcompat = { group = "androidx.appcompat", name = "appcompat", version.ref = "appcompat" }
material = { group = "com.google.android.material", name = "material", version.ref = "material" }
androidx-activity = { group = "androidx.activity", name = "activity", version.ref = "activity" }
androidx-constraintlayout = { group = "androidx.constraintlayout", name = "constraintlayout", version.ref = "constraintlayout" }

androidx-retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
androidx-converter-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "retrofit" }
androidx-gson = { group = "com.google.code.gson", name = "gson", version.ref = "gson" }

kotlinx-coroutines-android = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-android", version.ref = "coroutinesAndroid" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
```
