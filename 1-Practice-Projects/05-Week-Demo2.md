# MainActivity
```kotlin
package com.example.a05_week_demo2.View

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.hilt.navigation.compose.hiltViewModel
import com.example.a05_week_demo2.ViewModel.WeatherViewModel
import com.example.a05_week_demo2.ui.theme._05WeekDemo2Theme
import dagger.hilt.android.AndroidEntryPoint
import androidx.compose.material3.Button
import androidx.compose.material3.TextField
import androidx.compose.runtime.rememberCoroutineScope
import com.example.a05_week_demo2.ViewModel.WeatherUiState
import kotlinx.coroutines.launch

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            MainScreen()
        }
    }
}

@Composable
fun MainScreen(viewModel: WeatherViewModel = hiltViewModel()) {
    _05WeekDemo2Theme() {
        Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
            val uiState by viewModel.uiState.collectAsState()
            WeatherScreen(
                uiState = uiState,
                updateCityInput = {
                    viewModel.updateCityInput(it)
                },
                loadWeather = {
                    viewModel.loadWeather(it)
                },
                modifier = Modifier.padding(innerPadding)
            )
        }
    }
}

@Composable
fun WeatherScreen(
    uiState: WeatherUiState,
    updateCityInput: ((String) -> Unit)? = null,
    loadWeather: ((String) -> Unit)? = null,
    modifier: Modifier = Modifier,
) {

    Surface(
        modifier = modifier.fillMaxSize(),
        color = MaterialTheme.colorScheme.background
    ) {
        when {
            uiState.isLoading -> {
                // 加载状态
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    CircularProgressIndicator(color = MaterialTheme.colorScheme.primary)
                }
            }

            uiState.errorMessage != null -> {
                // 错误状态
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    Text(
                        text = uiState.errorMessage ?: "未知错误",
                        color = MaterialTheme.colorScheme.error,
                        style = MaterialTheme.typography.bodyLarge
                    )
                }
            }

            else -> {
                // 内容布局（包含输入框和查询按钮）
                Column(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(24.dp),
                    horizontalAlignment = Alignment.CenterHorizontally,
                    verticalArrangement = Arrangement.Top
                ) {
                    // 输入框
                    TextField(
                        value = uiState.cityInput, // 将状态的当前值显示在输入框中
                        onValueChange = {
                            updateCityInput?.invoke(it)
                        },
                        label = { Text("请输入城市名") },
                        modifier = Modifier.fillMaxWidth()
                    )
                    // 查询按钮
                    Button(
                        onClick = { loadWeather?.invoke((uiState.cityInput)) },
                        modifier = Modifier.padding(top = 16.dp)
                    ) {
                        Text("查询天气")
                    }
                    // 天气数据展示
                    Column(
                        modifier = Modifier.padding(top = 32.dp),
                        horizontalAlignment = Alignment.CenterHorizontally
                    ) {
                        Text(
                            text = "城市: ${uiState.weatherData.city}",
                            style = MaterialTheme.typography.headlineMedium,
                            modifier = Modifier.padding(bottom = 16.dp)
                        )
                        Text(
                            text = "温度: ${uiState.weatherData.tempC} °C",
                            style = MaterialTheme.typography.headlineSmall,
                            modifier = Modifier.padding(bottom = 12.dp)
                        )
                        Text(
                            text = "天气状况: ${uiState.weatherData.weatherDesc.firstOrNull()?.value ?: ""}",
                            style = MaterialTheme.typography.bodyLarge,
                            modifier = Modifier.padding(bottom = 12.dp)
                        )
                        Text(
                            text = "能见度: ${uiState.weatherData.visibility}",
                            style = MaterialTheme.typography.bodyLarge
                        )
                    }
                }
            }
        }
    }
}

// 顶级预览函数
@Preview(showBackground = true)
@Composable
fun WeatherScreenPreview() {
    _05WeekDemo2Theme {
        WeatherScreen(WeatherUiState())
    }
}
```
# ViewModel
```kotlin
package com.example.a05_week_demo2.ViewModel

import android.util.Log
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.rememberCoroutineScope
import androidx.compose.runtime.setValue
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.a05_week_demo2.Model.WeatherRepository
import com.example.a05_week_demo2.Model.data.WeatherDesc
import com.example.a05_week_demo2.Model.data.WeatherItem
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import javax.inject.Inject

sealed interface WeatherState{
    object Loading: WeatherState
    data class Success(val weatherData: WeatherItem): WeatherState
    data class Error(val message: String): WeatherState
    object Idle: WeatherState
}

data class WeatherUiState(
    val weatherState: WeatherState = WeatherState.Idle,
    val cityInput: String = ""
){
    // 提供便捷的 computed properties，保持向后兼容
    val isLoading: Boolean
        get() = weatherState is WeatherState.Loading

    val errorMessage: String?
        get() = when (weatherState) {
            is WeatherState.Error -> weatherState.message
            else -> null
        }

    val weatherData: WeatherItem
        get() = when (weatherState) {
            is WeatherState.Success -> weatherState.weatherData
            else -> WeatherItem() // 或者提供一个默认的空对象
        }
}

@HiltViewModel
class WeatherViewModel @Inject constructor(private val repository: WeatherRepository): ViewModel(){
    // 定义私有状态流
    private val _uiState = MutableStateFlow(WeatherUiState())
    val uiState = _uiState.asStateFlow()

    init {
        loadWeather("Beijing")
    }

    // 更新城市输入
    fun updateCityInput(city: String){
        _uiState.value = _uiState.value.copy(
            cityInput = city,
        )
    }
    // 加载天气数据
    fun loadWeather(city: String? = null){
        val targetCity = city ?: _uiState.value.cityInput
        if(targetCity.isBlank()){
            _uiState.value = _uiState.value.copy(
                weatherState = WeatherState.Error("请输入城市名全拼")
            )
            return
        }

        viewModelScope.launch {
            try {
                // 开始加载
                _uiState.value = _uiState.value.copy(
                    weatherState = WeatherState.Loading
                )

                val data = repository.getWeatherData(targetCity)

                _uiState.value = _uiState.value.copy(
                    weatherState = WeatherState.Success(data)
                )

            }catch (e: Exception){
                Log.d("WeatherViewModel", "加载天气数据失败", e)
                _uiState.value = _uiState.value.copy(
                    weatherState = WeatherState.Error("加载失败: ${e.message ?: "未知错误"}")
                )
            }
        }
    }
}
```
# RetrofitClient
```kotlin
package com.example.a05_week_demo2.Model

import okhttp3.OkHttpClient
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitClient {
    val BASE_URL = "https://whyta.cn/"

    val okHttpClient = OkHttpClient.Builder().build()

    val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .client(okHttpClient)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    val weatherApiService: WeatherApiService by lazy {
        retrofit.create(WeatherApiService::class.java)
    }
}
```
# WeatherApiService
```kotlin
package com.example.a05_week_demo2.Model

import com.example.a05_week_demo2.Model.data.WeatherApiResponse
import retrofit2.http.GET
import retrofit2.http.Query

interface WeatherApiService{
    @GET("api/tianqi")
    suspend fun getWeatherInfo(
        @Query("key") key: String = "738b541a5f7a",
        @Query("city") city: String
    ): WeatherApiResponse
}
```
# WeatherRepository
```kotlin
package com.example.a05_week_demo2.Model

import android.util.Log
import com.example.a05_week_demo2.Model.data.WeatherApiResponse
import com.example.a05_week_demo2.Model.data.WeatherItem
import javax.inject.Inject

interface WeatherRepository{
    suspend fun getWeatherData(city: String): WeatherItem
}

class WeatherRepositoryImpl @Inject constructor(
    private val weatherApiService: WeatherApiService
): WeatherRepository{
    override suspend fun getWeatherData(city: String): WeatherItem {
        return try {
            val response: WeatherApiResponse = weatherApiService.getWeatherInfo(city = city)
            if(response.message == "success"){
                val data = response.weatherItem ?: throw IllegalArgumentException("API返回结果为空")
                data
            }else{
                throw Exception("API返回失败: ${response.message}")
            }
        }catch (e: Exception){
            Log.d("getWeatherData", "获取数据失败")
            throw e
        }
    }
}
```
# WeatherApiResponse
```kotlin
package com.example.a05_week_demo2.Model.data

import com.google.gson.annotations.SerializedName

data class WeatherApiResponse(
    @SerializedName("status")
    val status: String?,
    @SerializedName("message")
    val message: String?,
    @SerializedName("data")
    val weatherItem: WeatherItem?
)
```
# WeatherDesc
```kotlin
package com.example.a05_week_demo2.Model.data

import com.google.gson.annotations.SerializedName

data class WeatherDesc (
    @SerializedName("value")
    val value: String? = ""
)
```

# WeatherItem
```kotlin
package com.example.a05_week_demo2.Model.data

import com.google.gson.annotations.SerializedName

data class WeatherItem(
    @SerializedName("city")
    val city: String? = "",
    @SerializedName("temp_C")
    val tempC: String? = "",
    @SerializedName("visibility")
    val visibility: String? = "",
    @SerializedName("weatherDesc")
    val weatherDesc: List<WeatherDesc> = emptyList()
)
```
# AppModule
```kotlin
package com.example.a05_week_demo2.di

import com.example.a05_week_demo2.Model.RetrofitClient
import com.example.a05_week_demo2.Model.WeatherApiService
import com.example.a05_week_demo2.Model.WeatherRepository
import com.example.a05_week_demo2.Model.WeatherRepositoryImpl
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Singleton
    @Provides
    fun provideApiService(): WeatherApiService{
        return RetrofitClient.weatherApiService
    }

    @Singleton
    @Provides
    fun provideWeatherRepository(
        weatherApiService: WeatherApiService
    ): WeatherRepository{
        return WeatherRepositoryImpl(weatherApiService)
    }
}
```

# MyApplication
```kotlin
package com.example.a05_week_demo2

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class MyApplication: Application(){
    override fun onCreate(){
        super.onCreate();
    }
}
```
