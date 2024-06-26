# How to fetch data from free API with your Android Compose Kotlin application

In this example I would explain you how to send a GET request to a public API endpoint from your Android Compose application to retrieve data from a server

This is the public API endpoint we are going to interrogate

https://jsonplaceholder.typicode.com/todos

## 1. Create a new Compose project in Android Studio

We run Android Studio and we press the **New Project** button

![image](https://github.com/luiscoco/Android_Kotlin_lesson2_MyFirstComposeApp-advanced-with-fetching-data-/assets/32194879/77e70d6c-0f24-407c-9e59-cf72aa9dd947)

We select the **Empty Activity** template

![image](https://github.com/luiscoco/Android_Kotlin_lesson2_MyFirstComposeApp-advanced-with-fetching-data-/assets/32194879/9e096923-b0a6-48c5-bb7a-8cacb1e7d6d8)

We input the required data: project name, location, package name, minimum SDK, build configuration language

![image](https://github.com/luiscoco/Android_Kotlin_lesson2_MyFirstComposeApp-advanced-with-fetching-data-/assets/32194879/a099bec2-88d3-4c6b-bf2b-61d041299ae5)

## 2. Project Folders and Files structure

![image](https://github.com/luiscoco/Android_Kotlin_lesson2_MyFirstComposeApp-advanced-with-fetching-data-/assets/32194879/d5b49b4c-a05d-424b-b232-f4ffcef1b7ed)

## 3. Project Dependencies

![image](https://github.com/luiscoco/Android_Kotlin_lesson2_MyFirstComposeApp-advanced-with-fetching-data-/assets/32194879/193265b9-8143-4e1f-9193-29d5c25016fd)

**build.gradle.kts**(Project:MyFetchApplication)

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.jetbrains.kotlin.android) apply false
    kotlin("jvm") version "1.6.10"
    kotlin("plugin.serialization") version "1.6.10"
    id("org.jetbrains.compose") version "1.6.10"
}
```

**build.gradle.kts**(Module:app)

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.jetbrains.kotlin.android)
}

android {
    namespace = "com.example.myfetchapplication"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.myfetchapplication"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        vectorDrawables {
            useSupportLibrary = true
        }
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
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
    buildFeatures {
        compose = true
    }
    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.1"
    }
    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
}

dependencies {
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.activity.compose)
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.ui)
    implementation(libs.androidx.ui.graphics)
    implementation(libs.androidx.ui.tooling.preview)
    implementation(libs.androidx.material3)
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.0-RC")
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
    implementation("com.squareup.retrofit2:converter-gson:2.11.0")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.4.0")
    implementation("androidx.compose.runtime:runtime-livedata:1.2.0-alpha01")
    implementation(libs.androidx.runtime.livedata)

    // Required for previews
    implementation("androidx.compose.ui:ui-tooling-preview:1.4.0")
    implementation("androidx.compose.ui:ui-tooling:1.4.0")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.5.1")
    implementation("androidx.compose.material3:material3:1.0.1")

    // Test dependencies
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
    androidTestImplementation(platform(libs.androidx.compose.bom))
    androidTestImplementation(libs.androidx.ui.test.junit4)

    // Debug dependencies
    debugImplementation(libs.androidx.ui.tooling)
    debugImplementation(libs.androidx.ui.test.manifest)
}
```

**settings.gradle.kts**

```kotlin
pluginManagement {
    repositories {
        google {
            content {
                includeGroupByRegex("com\\.android.*")
                includeGroupByRegex("com\\.google.*")
                includeGroupByRegex("androidx.*")
            }
        }
        mavenCentral()
        gradlePluginPortal()
        maven("https://maven.pkg.jetbrains.space/public/p/compose/dev")
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven("https://maven.pkg.jetbrains.space/public/p/compose/dev")
    }
}

rootProject.name = "MyFetchApplication"
include(":app")
```

## 4. Add Internet Permission 

I it very important to highlight we added the following code in order to grant the application permission to access internet

```
<!-- Internet permission to allow network requests -->
<uses-permission android:name="android.permission.INTERNET" />
```

This is the full code

**app/manifests/AndroidManifest.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- Internet permission to allow network requests -->
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyFetchApplication"
        tools:targetApi="31">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="@string/app_name"
            android:theme="@style/Theme.MyFetchApplication">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

## 5. Data Model

**app/kotlin+java/com.example.myfetchapplication/data/model/Todo.kt**

```kotlin
package com.example.myfetchapplication.data.model

import kotlinx.serialization.Serializable

@Serializable
data class Todo(
    val userId: Int,
    val id: Int,
    val title: String,
    val completed: Boolean
)
```

## 6. API Service

**app/kotlin+java/com.example.myfetchapplication/data/network/ApiService.kt**

```kotlin
package com.example.myfetchapplication.data.network

import com.example.myfetchapplication.data.model.Todo
import retrofit2.http.GET

interface ApiService {
    @GET("todos")
    suspend fun getTodos(): List<Todo>
}
```

**app/kotlin+java/com.example.myfetchapplication/data/network/RetrofitInstance.kt**

```kotlin
package com.example.myfetchapplication.data.network

import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitInstance {
    private val retrofit by lazy {
        Retrofit.Builder()
            .baseUrl("https://jsonplaceholder.typicode.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    val api: ApiService by lazy {
        retrofit.create(ApiService::class.java)
    }
}
```

## 7. Data Repository

**app/kotlin+java/com.example.myfetchapplication/repository/TodoRepository.kt**

```kotlin
package com.example.myfetchapplication.repository

import com.example.myfetchapplication.data.network.RetrofitInstance

class TodoRepository {
    suspend fun getTodos() = RetrofitInstance.api.getTodos()
}
```

**app/kotlin+java/com.example.myfetchapplication/repository/PreviewTodoRepository.kt**

```kotlin
package com.example.myfetchapplication.repository

import com.example.myfetchapplication.data.model.Todo

class PreviewTodoRepository : TodoRepository() {
    override suspend fun getTodos(): List<Todo> {
        // Return mock data
        return listOf(
            Todo(
                id = 1,
                userId = 1,
                title = "Preview Todo 1",
                completed = false
            ),
            Todo(
                id = 2,
                userId = 2,
                title = "Preview Todo 2",
                completed = true
            ),
            Todo(
                id = 3,
                userId = 3,
                title = "Preview Todo 3",
                completed = false
            )
        )
    }
}
```

## 8. User Interface (UI) Screens

### 8.1. Greeting

We firt desing a **Card** 

![image](https://github.com/luiscoco/Android_Kotlin_lesson2_MyFirstComposeApp-advanced-with-fetching-data-/assets/32194879/f2c55330-cd05-48fa-a796-c99e679f8170)

The **Card** contains the **CardContent**

![image](https://github.com/luiscoco/Android_Kotlin_lesson2_MyFirstComposeApp-advanced-with-fetching-data-/assets/32194879/edad4b50-c1ce-4a3c-997b-b8d9f837e6d2)

**app/kotlin+java/com.example.myfetchapplication/ui/screens/Greeting.kt**

```kotlin
package com.example.myfetchapplication.ui.screens

import androidx.compose.animation.animateContentSize
import androidx.compose.animation.core.Spring
import androidx.compose.animation.core.spring
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.padding
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.KeyboardArrowDown
import androidx.compose.material.icons.filled.KeyboardArrowUp
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.IconButton
import androidx.compose.material3.Text
import androidx.compose.runtime.saveable.rememberSaveable
import androidx.compose.ui.res.stringResource
import com.example.myfetchapplication.R
import com.example.myfetchapplication.data.model.Todo

@Composable
fun Greeting(todo: Todo, modifier: Modifier = Modifier) {
    Card(
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.primary
        ),
        modifier = modifier.padding(vertical = 4.dp, horizontal = 8.dp)
    ) {
        CardContent(todo)
    }
}

@Composable
fun CardContent(todo: Todo) {
    var expanded by rememberSaveable { mutableStateOf(false) }

    Row(
        modifier = Modifier
            .padding(12.dp)
            .animateContentSize(
                animationSpec = spring(
                    dampingRatio = Spring.DampingRatioMediumBouncy,
                    stiffness = Spring.StiffnessLow
                )
            )
    ) {
        Column(
            modifier = Modifier
                .weight(1f)
                .padding(12.dp)
        ) {
            Text(text = "Todo ID: ${todo.id}")
            Text(
                text = todo.title,
                style = MaterialTheme.typography.headlineMedium.copy(
                    fontWeight = FontWeight.ExtraBold
                )
            )
            if (expanded) {
                Text(
                    text = "Completed: ${todo.completed}",
                )
            }
        }
        IconButton(onClick = { expanded = !expanded }) {
            Icon(
                imageVector = if (expanded) Icons.Filled.KeyboardArrowUp else Icons.Filled.KeyboardArrowDown,
                contentDescription = if (expanded) {
                    stringResource(id = R.string.show_less)
                } else {
                    stringResource(id = R.string.show_more)
                }
            )
        }
    }
}
```

### 8.2. Greeting Preview

**app/kotlin+java/com.example.myfetchapplication/ui/screens/previews/GreetingPreview.kt**

```kotlin
package com.example.myfetchapplication.ui.screens.previews

import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import com.example.myfetchapplication.data.model.Todo
import com.example.myfetchapplication.ui.screens.CardContent
import com.example.myfetchapplication.ui.screens.Greeting

@Preview(showBackground = true)
@Composable
fun GreetingPreview() {
    Greeting(
        todo = Todo(
            id = 1,
            userId = 1,
            title = "Example Todo",
            completed = false
        ),
        modifier = Modifier
    )
}

@Preview(showBackground = true)
@Composable
fun CardContentPreview() {
    CardContent(
        todo = Todo(
            id = 1,
            userId = 1,
            title = "Example Todo",
            completed = false
        )
    )
}
```

### 8.3. Greetings

![image](https://github.com/luiscoco/Android_Kotlin_lesson2_MyFirstComposeApp-advanced-with-fetching-data-/assets/32194879/1b256a1d-cb1c-450e-97ef-c383dad5e082)

**app/kotlin+java/com.example.myfetchapplication/ui/screens/Greetings.kt**

```kotlin
package com.example.myfetchapplication.ui.screens

import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.compose.foundation.lazy.items
import com.example.myfetchapplication.data.model.Todo

@Composable
fun Greetings(modifier: Modifier = Modifier, todos: List<Todo>) {
    LazyColumn(modifier = modifier.padding(vertical = 4.dp)) {
        items(items = todos) { todo ->
            Greeting(todo = todo)
        }
    }
}
```

### 8.4. Greetings Preview

**app/kotlin+java/com.example.myfetchapplication/ui/screens/preview/GreetingsPreview.kt**

```kotlin
package com.example.myfetchapplication.ui.screens.previews

import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import com.example.myfetchapplication.data.model.Todo
import com.example.myfetchapplication.ui.screens.Greetings

@Preview(showBackground = true)
@Composable
fun GreetingsPreview() {
    val sampleTodos = listOf(
        Todo(
            id = 1,
            userId = 1,
            title = "Example Todo 1",
            completed = false
        ),
        Todo(
            id = 2,
            userId = 2,
            title = "Example Todo 2",
            completed = true
        ),
        Todo(
            id = 3,
            userId = 3,
            title = "Example Todo 3",
            completed = false
        )
    )

    Greetings(
        modifier = Modifier,
        todos = sampleTodos
    )
}
```

### 8.5. TodoScreen

![image](https://github.com/luiscoco/Android_Kotlin_lesson2_MyFirstComposeApp-advanced-with-fetching-data-/assets/32194879/7aab6798-7a01-4f10-8069-2a8455ed35b9)

**app/kotlin+java/com.example.myfetchapplication/ui/screens/TodoScreen.kt**

```kotlin
package com.example.myfetchapplication.ui.screens

import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.livedata.observeAsState
import androidx.lifecycle.viewmodel.compose.viewModel
import com.example.myfetchapplication.viewmodel.TodoViewModel

@Composable
fun TodoScreen(todoViewModel: TodoViewModel = viewModel()) {
    val todos by todoViewModel.todos.observeAsState(emptyList())
    Greetings(todos = todos)
}
```

### 8.5. TodoScreen Preview

**app/kotlin+java/com.example.myfetchapplication/ui/screens/TodoScreenPreview.kt**

```kotlin
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.livedata.observeAsState
import com.example.myfetchapplication.ui.screens.previews.GreetingsPreview
import com.example.myfetchapplication.viewmodel.PreviewTodoViewModel

@Preview(showBackground = true)
@Composable
fun TodoScreenPreview() {
    val todoViewModel = PreviewTodoViewModel()
    val todos by todoViewModel.todos.observeAsState(emptyList())
    GreetingsPreview()
}
```

### 8.7. Theme

**app/kotlin+java/com.example.myfetchapplication/ui/theme/Color.kt**

```kotlin
package com.example.myfetchapplication.ui.theme

import androidx.compose.ui.graphics.Color

val Purple80 = Color(0xFFD0BCFF)
val PurpleGrey80 = Color(0xFFCCC2DC)
val Pink80 = Color(0xFFEFB8C8)

val Purple40 = Color(0xFF6650a4)
val PurpleGrey40 = Color(0xFF625b71)
val Pink40 = Color(0xFF7D5260)
```

**app/kotlin+java/com.example.myfetchapplication/ui/theme/Type.kt**

```kotlin
package com.example.myfetchapplication.ui.theme

import androidx.compose.material3.Typography
import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.sp

// Set of Material typography styles to start with
val Typography = Typography(
    bodyLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp
    )
)
```

**app/kotlin+java/com.example.myfetchapplication/ui/theme/Type.kt**

```kotlin
package com.example.myfetchapplication.ui.theme

import android.app.Activity
import android.os.Build
import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.darkColorScheme
import androidx.compose.material3.dynamicDarkColorScheme
import androidx.compose.material3.dynamicLightColorScheme
import androidx.compose.material3.lightColorScheme
import androidx.compose.runtime.Composable
import androidx.compose.ui.platform.LocalContext

private val DarkColorScheme = darkColorScheme(
    primary = Purple80,
    secondary = PurpleGrey80,
    tertiary = Pink80
)

private val LightColorScheme = lightColorScheme(
    primary = Purple40,
    secondary = PurpleGrey40,
    tertiary = Pink40
)

@Composable
fun MyFetchApplicationTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }

        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}
```

## 9. ViewModel

**app/kotlin+java/com.example.myfetchapplication/viewmodel/TodoViewModel.kt**

```kotlin
package com.example.myfetchapplication.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.launch
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import com.example.myfetchapplication.data.model.Todo
import com.example.myfetchapplication.repository.TodoRepository

class TodoViewModel : ViewModel() {
    private val repository = TodoRepository()
    private val _todos = MutableLiveData<List<Todo>>()
    val todos: LiveData<List<Todo>> get() = _todos

    init {
        fetchTodos()
    }

    private fun fetchTodos() {
        viewModelScope.launch {
            val fetchedTodos = repository.getTodos()
            _todos.value = fetchedTodos
        }
    }
}
```

**app/kotlin+java/com.example.myfetchapplication/viewmodel/PreviewTodoViewModel.kt**

```kotlin
package com.example.myfetchapplication.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.launch
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import com.example.myfetchapplication.data.model.Todo
import com.example.myfetchapplication.repository.TodoRepository
import com.example.myfetchapplication.repository.PreviewTodoRepository

class PreviewTodoViewModel : ViewModel() {
    private val repository = PreviewTodoRepository()
    private val _todos = MutableLiveData<List<Todo>>()
    val todos: LiveData<List<Todo>> get() = _todos

    init {
        fetchTodos()
    }

    private fun fetchTodos() {
        viewModelScope.launch {
            val fetchedTodos = repository.getTodos()
            _todos.value = fetchedTodos
        }
    }
}
```

## 10. MainActivity

**app/kotlin+java/com.example.myfetchapplication/MainActivity.kt**

```kotlin
package com.example.myfetchapplication

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import com.example.myfetchapplication.ui.screens.TodoScreen
import com.example.myfetchapplication.ui.theme.MyFetchApplicationTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyFetchApplicationTheme {
                TodoScreen()
            }
        }
    }
}
```

## 11.  How to run the application

As prerequesite we have to install the **Android Virtual Device (AVD)** in Android Studio

We fist press the **Run app** button and the **Running Devices** button

![image](https://github.com/luiscoco/Android_Kotlin_lesson2_MyFirstComposeApp-advanced-with-fetching-data-/assets/32194879/5218e13d-10af-40ed-9aa6-dae77d19e3e6)

We can verify the application running

![image](https://github.com/luiscoco/Android_Kotlin_lesson2_MyFirstComposeApp-advanced-with-fetching-data-/assets/32194879/6f98c540-fc95-4100-8ad1-1adb54496dcb)


