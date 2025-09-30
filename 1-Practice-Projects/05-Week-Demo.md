```kotlin
package com.example.a05_week_demo

import android.content.res.Configuration
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.Image
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.text.style.TextDecoration
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.example.a05_week_demo.ui.theme._05WeekDemoTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            _05WeekDemoTheme {
                Surface(modifier = Modifier.fillMaxSize()) {
                    Conversation(sampleMessages)
                }
            }
        }
    }
}

data class Message(val author: String, val body: String)


@Composable
fun MessageCard(msg: Message){
    Row(Modifier.padding(8.dp)){//水平排列
        Image(
            painterResource(R.drawable.ic_launcher_foreground),
            "Android image",
            Modifier.size(80.dp).clip(CircleShape)
        )
        Spacer(Modifier.width(8.dp))

        var isExpanded by remember{mutableStateOf(false)}

        Column(modifier = Modifier.clickable{isExpanded = !isExpanded}) {//垂直排列
            Text(
                text = msg.author,
                color = MaterialTheme.colorScheme.secondary,
                style = MaterialTheme.typography.titleSmall,

            )
            Spacer(Modifier.height(2.dp))

            Surface(shape = MaterialTheme.shapes.medium, shadowElevation = 1.dp){
                Text(
                    text = msg.body,
                    modifier = Modifier.padding(all = 4.dp),
                    style = MaterialTheme.typography.bodyMedium,
                    maxLines = if(isExpanded)Int.MAX_VALUE else 1,
                    textDecoration = TextDecoration.Underline,
                    softWrap = true, // 是否自动换行
                )
            }
        }
    }
}

@Preview(name = "Light Mode")
@Preview(
    uiMode = Configuration.UI_MODE_NIGHT_YES,
    showBackground = true,
    name = "Dark Mode")
@Composable
fun PreViewMessageCard(){
    _05WeekDemoTheme {
        Surface {
            MessageCard(
                msg = Message("Lexi", "Take a look at Jetpack Compose, it's great!")
            )
        }
    }
}
// 生成10条示例聊天数据（包含长文本）
val sampleMessages = listOf(
    Message("Alice", "嗨，早上好，现在我真的需要休息一下，为了下面能够更好的工作也和生活"),
    Message("Bob", "是啊，适合出去走走。我计划下午去爬山，有谁想一起吗？我们可以去西山，那边的风景很好，而且现在这个季节人也不多。"),
    Message("Charlie", "有人想一起去喝咖啡吗？我知道一家新开的咖啡馆，环境很不错，咖啡也很好喝。他们还有露天座位，可以一边享受阳光一边聊天。"),
    Message("David", "好主意！下午3点怎么样？那个时间点人应该不多，我们可以安静地聊聊天。不过我得先完成手头的工作报告，希望能在3点前搞定。"),
    Message("Eva", "我可以参加，在哪里见面？请告诉我具体地址，或者发个定位给我。我对那边不太熟悉，可能需要提前出发找一下位置。"),
    Message("Frank", "星巴克中央公园店吧，那里位置比较中心，大家都方便找到。而且他们最近推出了新的春季特调咖啡，听说味道很不错，我们可以尝尝看。"),
    Message("Grace", "抱歉，我今天有事去不了。我有个重要的会议要参加，估计会开到很晚。希望你们玩得开心，下次有机会我一定参加！"),
    Message("Henry", "没关系，下次再约。其实我们可以建立一个固定的周末聚会机制，这样大家都能提前安排时间。比如每个月第一个周末聚一次？"),
    Message("Ivy", "我已经到了，穿蓝色外套，坐在靠窗的位置。这里的环境确实不错，音乐很轻柔，座位也很舒适。你们到了之后直接进来找我就行。"),
    Message("Jack", "看到你了，马上到！我刚刚停好车，正在过马路。今天路上有点堵，不好意思让大家久等了。我带了点小点心过来一起分享。"),
    Message("Henry", "没关系，下次再约。其实我们可以建立一个固定的周末聚会机制，这样大家都能提前安排时间。比如每个月第一个周末聚一次？"),
    Message("Ivy", "我已经到了，穿蓝色外套，坐在靠窗的位置。这里的环境确实不错，音乐很轻柔，座位也很舒适。你们到了之后直接进来找我就行。"),
    Message("Jack", "看到你了，马上到！我刚刚停好车，正在过马路。今天路上有点堵，不好意思让大家久等了。我带了点小点心过来一起分享。")
)
@Composable
fun Conversation(msgs: List<Message>){
    LazyColumn{
        items(msgs){ msg ->
            MessageCard(msg)
        }
    }
}
@Preview
@Composable
fun PreViewConversation(){
    _05WeekDemoTheme{
        Conversation(sampleMessages)
    }
}
```
