# View
### MainActivity
```kotlin
package com.example.a03_week_demo2.com.example.a03_week_demo2.View

import android.content.Intent
import android.net.Uri
import android.os.Bundle
import android.view.View
import android.widget.Toast
import androidx.activity.enableEdgeToEdge
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import androidx.lifecycle.lifecycleScope
import androidx.recyclerview.widget.LinearLayoutManager
import com.example.a03_week_demo2.R
import com.example.a03_week_demo2.com.example.a03_week_demo2.ViewModel.NewsViewModel
import com.example.a03_week_demo2.databinding.ActivityMainBinding
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private val viewModel: NewsViewModel by viewModels()
    private lateinit var adapter: MyAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // 使用ViewBinding替换findViewById--------
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        // --------------------------------------

        enableEdgeToEdge()
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }

        initRecyclerView()
        observeViewModel()
    }

    private fun initRecyclerView() {
        adapter = MyAdapter(emptyList()) { newsItem ->
            openNewsUrl(newsItem.url)
        }

        binding.recyclerView.layoutManager = LinearLayoutManager(this)
        binding.recyclerView.adapter = adapter
    }

    private fun observeViewModel(){
        lifecycleScope.launch{
            viewModel.uiState.collect { state ->
                when(state){
                    is NewsViewModel.UiState.Loading -> {
                        // 显示加载过程
                        binding.progressBar.visibility = View.VISIBLE
                    }
                    is NewsViewModel.UiState.Success -> {
                        // 隐藏加载过程
                        binding.progressBar.visibility = View.GONE
                        // 更新数据
                        adapter.updateData(state.newsList)
                    }
                    is NewsViewModel.UiState.Error -> {
                        // 隐藏加载过程
                        binding.progressBar.visibility = View.GONE
                        // 短暂
                        Toast.makeText(this@MainActivity, state.message,Toast.LENGTH_SHORT).show()
                    }
                }
            }
        }
    }

    private fun openNewsUrl(url: String?) {
        if (url.isNullOrEmpty()) {
            Toast.makeText(this, "链接无效", Toast.LENGTH_SHORT).show()
            return
        }
        val intent = Intent(Intent.ACTION_VIEW, Uri.parse(url))
        startActivity(intent)
    }
}
```
### MyAdapter
```kotlin
package com.example.a03_week_demo2.com.example.a03_week_demo2.View

import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.RecyclerView
import com.bumptech.glide.Glide
import com.example.a03_week_demo2.R
import com.example.a03_week_demo2.com.example.a03_week_demo2.View.MyHolder
import com.example.a03_week_demo2.Model.data.NewsItem

class MyAdapter(
    private var dataList: List<NewsItem>,
    private val onItemClick: (NewsItem) -> Unit
) : RecyclerView.Adapter<MyHolder>() {

    // 更新数据的方法
    fun updateData(newList: List<NewsItem>) {
        dataList = newList
        notifyDataSetChanged()
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyHolder {
        val itemView = LayoutInflater.from(parent.context).inflate(R.layout.newsitem, parent, false)
        return MyHolder(itemView)
    }

    override fun onBindViewHolder(holder: MyHolder, position: Int) {
        val currentItem = dataList[position]

        // 图片加载
        if (currentItem.picUrl.isNullOrEmpty()) {
            Glide.with(holder.itemView.context)
                .load(R.drawable.ic_launcher_background)
                .into(holder.image)
        } else {
            Glide.with(holder.itemView.context)
                .load(currentItem.picUrl)
                .placeholder(R.drawable.ic_launcher_background)
                .error(R.drawable.ic_launcher_background)
                .into(holder.image)
        }

        holder.title.text = currentItem.title
        holder.source.text = currentItem.source
        holder.ctime.text = currentItem.ctime

        // 点击事件
        holder.itemView.setOnClickListener {
            onItemClick(currentItem)
        }
    }

    override fun getItemCount(): Int = dataList.size
}
```
### MyHolder
```kotlin
package com.example.a03_week_demo2.com.example.a03_week_demo2.View

import android.view.View
import android.widget.ImageView
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.example.a03_week_demo2.R

class MyHolder(view: View): RecyclerView.ViewHolder(view){
    val image: ImageView = itemView.findViewById(R.id.imageView)
    val title: TextView = itemView.findViewById(R.id.hinttitle)
    val source: TextView = itemView.findViewById(R.id.hintsource)
    val ctime: TextView = itemView.findViewById(R.id.hintCtime)
}
```
# ViewModel
### NewsViewModel
```kotlin
package com.example.a03_week_demo2.com.example.a03_week_demo2.ViewModel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.a03_week_demo2.com.example.a03_week_demo2.data.RetrofitClient
import com.example.a03_week_demo2.Model.data.NewsItem
import com.example.a03_week_demo2.com.example.a03_week_demo2.data.NewsRepository
import com.example.a03_week_demo2.com.example.a03_week_demo2.data.NewsRepositoryImpl
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch

class NewsViewModel : ViewModel() {

    // 密封类定义UI状态
    sealed class UiState {
        object Loading : UiState()
        data class Success(val newsList: List<NewsItem>) : UiState()
        data class Error(val message: String) : UiState()
    }

    // 私有可变的StateFlow
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    // 对外暴露只读的StateFlow
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()

    init {
        loadNews()
    }

    fun loadNews() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            try {
                val newsList = NewsRepositoryImpl.create().getNewsList()
                _uiState.value = UiState.Success(newsList)
            } catch (e: Exception) {
                _uiState.value = UiState.Error("网络错误: ${e.message}")
            }
        }
    }

    // 刷新数据的方法
    fun refreshNews() {
        loadNews()
    }
}
```
# Model
### NewsRepository
```kotlin
package com.example.a03_week_demo2.com.example.a03_week_demo2.data

import com.example.a03_week_demo2.Model.data.ApiResponse
import com.example.a03_week_demo2.Model.data.NewsItem

//接口不能包含实现
interface NewsRepository{
    suspend fun getNewsList(): List<NewsItem>
}

class NewsRepositoryImpl constructor(private val newsApiService: NewsApiService) : NewsRepository{

    // 从网络API获取数据
    override suspend fun getNewsList(): List<NewsItem>{
        val response: ApiResponse = newsApiService.getNewsList()
        if(response.code == 200){
            return response.result.newsList
        }else{
            throw Exception("API错误: ${response.msg}")
        }
    }

    // 伴生对象，是 Kotlin 实现 "静态成员" 功能的方式
    companion object {
        fun create() = NewsRepositoryImpl(RetrofitClient.newsApiService)
    }
}
```
### NewsApiService
```kotlin
package com.example.a03_week_demo2.com.example.a03_week_demo2.data

import com.example.a03_week_demo2.Model.data.ApiResponse
import retrofit2.http.GET
import retrofit2.http.Query

interface NewsApiService {
    @GET("api/tx/guonei")
    suspend fun getNewsList(
        @Query("key") key: String = "738b541a5f7a",
        @Query("num") num: Int = 10
    ): ApiResponse
}
```
### RetrofitClient
```kotlin
package com.example.a03_week_demo2.com.example.a03_week_demo2.data

import com.example.a03_week_demo2.com.example.a03_week_demo2.data.NewsApiService
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
### ApiResponse
```kotlin
package com.example.a03_week_demo2.Model.data

data class ApiResponse(
    val code: Int,
    val msg: String,
    val result: NewsResult
)
```
### NewsResult
```kotlin
package com.example.a03_week_demo2.Model.data

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
### NewsItem
```kotlin
package com.example.a03_week_demo2.Model.data

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
# Gradle
### build.gradle.kts Project
```kotlin
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.kapt) apply false
    alias(libs.plugins.hilt.android) apply false
}
```
### build.gradle.kts Module
```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.kapt)
    alias(libs.plugins.hilt.android)
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


    buildFeatures {
        viewBinding = true
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
    implementation(libs.androidx.glide)
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.appcompat)
    implementation(libs.material)
    implementation(libs.androidx.activity)
    implementation(libs.androidx.constraintlayout)
    implementation(libs.androidx.retrofit)
    implementation(libs.androidx.converter.gson)
    implementation(libs.androidx.gson)
    implementation(libs.androidx.recyclerview)
    testImplementation(libs.junit)
    implementation(libs.androidx.lifecycle.viewmodel.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)

    implementation(libs.hilt.android)
    kapt(libs.hilt.compiler)
    implementation(libs.hilt.navigation.fragment)
}
```
### libs.version.toml
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
recyclerview = "1.4.0"
glide = "4.16.0"
lifecycle = "2.8.2"
hilt = "2.48.1"
hiltNavigation = "1.0.0"

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
androidx-recyclerview = { group = "androidx.recyclerview", name = "recyclerview", version.ref = "recyclerview" }
androidx-glide = { group = "com.github.bumptech.glide", name = "glide", version.ref = "glide" }
androidx-lifecycle-viewmodel-ktx = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-ktx", version.ref = "lifecycle" }
androidx-lifecycle-runtime-ktx = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version.ref = "lifecycle" }
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
hilt-navigation-fragment = { group = "androidx.hilt", name = "hilt-navigation-fragment", version.ref = "hiltNavigation" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-kapt = { id = "org.jetbrains.kotlin.kapt", version.ref = "kotlin" }
hilt-android = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```
