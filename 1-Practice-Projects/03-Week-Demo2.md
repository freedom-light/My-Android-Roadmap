# MainActivity
```kotlin
package com.example.a03_week_demo2

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.util.Log
import android.widget.Toast
import androidx.lifecycle.lifecycleScope
import com.example.a03_week_demo2.com.example.a03_week_demo2.data.NewsItem
import kotlinx.coroutines.launch

// 如果你用了 RecyclerView，需要导入相关类（示例中先简化用 Toast/Log 展示）
// import androidx.recyclerview.widget.LinearLayoutManager
// import androidx.recyclerview.widget.RecyclerView

class MainActivity : AppCompatActivity() {
    // 1. 声明 UI 控件（根据你的 activity_main.xml 布局调整，这里示例常用控件）
    // private lateinit var rvNews: RecyclerView // 新闻列表RecyclerView
    // private lateinit var progressBar: View     // 加载进度条
    // private lateinit var tvError: View         // 错误提示文本

    // 2. 新闻列表数据（用于后续 UI 展示）
    private var newsDataList: List<NewsItem> = emptyList()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // 加载布局（确保 activity_main.xml 存在对应控件）
        setContentView(R.layout.activity_main)

        // 3. 绑定 UI 控件（根据你的布局ID修改，示例ID仅参考）
        // progressBar = findViewById(R.id.progress_bar)
        // tvError = findViewById(R.id.tv_error)
        // rvNews = findViewById(R.id.rv_news)

        // 4. 初始化 RecyclerView（如果用了列表展示，没用到可删除这部分）
        // initRecyclerView()

        // 5. 启动网络请求加载新闻数据
        loadNewsData()
    }

    /**
     * 初始化 RecyclerView（列表展示用，没用到可删除）
     */
    // private fun initRecyclerView() {
    //     rvNews.layoutManager = LinearLayoutManager(this)
    //     // 后续获取数据后设置适配器：rvNews.adapter = NewsAdapter(newsDataList)
    // }

    /**
     * 核心：发起网络请求获取新闻数据
     */
    private fun loadNewsData() {
        // 切换到协程（lifecycleScope 自动绑定Activity生命周期，避免内存泄漏）
        lifecycleScope.launch {
            try {
                // 1. 显示加载状态（让用户知道在加载）
                // progressBar.visibility = View.VISIBLE
                // tvError.visibility = View.GONE // 隐藏错误提示
                Log.d("MainActivity", "开始加载新闻数据...")

                // 2. 调用 Retrofit 接口请求数据（直接用你配置的默认参数 key/num）
                val response: ApiResponse = RetrofitClient.newsApiService.getNewsList()

                // 3. 处理 API 响应（关键：按你的 ApiResponse 结构解析）
                if (response.code == 200) { // 假设 code=200 代表请求成功
                    // 从 ApiResponse -> NewsResult -> newsList 拿到最终新闻列表
                    newsDataList = response.result.newsList

                    // 打印日志确认数据（方便调试）
                    Log.d("MainActivity", "请求成功！共获取 ${newsDataList.size} 条新闻")
                    newsDataList.forEachIndexed { index, item ->
                        Log.d("NewsItem-$index", "标题：${item.title} | 来源：${item.source}")
                    }

                    // 4. 更新 UI 展示数据（必须在主线程，runOnUiThread 确保主线程执行）
                    runOnUiThread {
                        // 隐藏加载进度
                        // progressBar.visibility = View.GONE

                        // 情况1：如果用 RecyclerView 展示列表
                        // rvNews.adapter = NewsAdapter(newsDataList) // 需自己实现 NewsAdapter

                        // 情况2：简化用 Toast 提示结果（没做列表时先用这个验证）
                        Toast.makeText(
                            this@MainActivity,
                            "加载成功！共 ${newsDataList.size} 条新闻",
                            Toast.LENGTH_SHORT
                        ).show()
                    }

                } else {
                    // API 返回错误（比如 code≠200，按你的 msg 提示）
                    val errorMsg = "API错误：${response.msg}（错误码：${response.code}）"
                    Log.e("MainActivity", errorMsg)
                    showErrorUI(errorMsg) // 显示错误UI
                }

            } catch (e: Exception) {
                // 5. 处理网络异常（比如没网、超时、解析错误等）
                val errorMsg = "网络请求失败：${e.message ?: "未知错误"}"
                Log.e("MainActivity", errorMsg, e) // 打印异常堆栈，方便调试
                showErrorUI(errorMsg) // 显示错误UI

            } finally {
                // 6. 最终：无论成功/失败，都隐藏加载进度
                // progressBar.visibility = View.GONE
                Log.d("MainActivity", "加载流程结束")
            }
        }
    }

    /**
     * 统一处理错误UI（避免重复代码）
     */
    private fun showErrorUI(errorMsg: String) {
        runOnUiThread {
            // 显示错误提示，隐藏列表（如果有）
            // tvError.visibility = View.VISIBLE
            // tvError.text = errorMsg
            // rvNews.visibility = View.GONE

            // 简化用 Toast 提示错误（没做错误文本时先用这个）
            Toast.makeText(this@MainActivity, errorMsg, Toast.LENGTH_LONG).show()
        }
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
