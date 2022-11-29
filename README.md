# tips_android_programing

## Room

### Declarate Database

``` kotlin

@Database(
    entities = [// list entity ],
    version = 1,
    exportSchema = false
)
@TypeConverters(Converts::class)
abstract class NurseDatabase: RoomDatabase() {
    abstract fun getObjectDAO(): ObjectDAO
}

```

### Declarate Dao

``` kotlin

@Dao
interface ObjectDAO {
   // declarate interfaces
}

```

This interfaces can operations basics as Insert, Deleter, Update or more Querys

``` kotlin

  @Insert(onConflict = OnConflictStrategy.REPLACE)
  suspend fun insertObject(item: Object): Long
    
  @Delete
  suspend fun deleterObject(item: Object)
  
  @Update
  suspend fun updateObject(item: Object)
  
  // deleter using id object
   @Query("DELETE FROM generic_table WHERE id=:idObject")
   suspend fun deleterObjectById(idObject: Int)
   
   // deleter passes list ids object
   @Query("DELETE FROM generic_table WHERE id IN (:listIdDeleter)")
   suspend fun deleterListObjectById(listIdDeleter: List<Long>)
   
   @Query("SELECT * FROM generic_table WHERE id=:id")
    suspend fun getObjectById(id: Long): Object?
```
### Declarate entity

``` kotlin


@Entity(tableName = "name_table")
data class Object(
    //any propertys...
    @PrimaryKey(autoGenerate = true)
    override val id: Long=0
)
```

### Get database with hilt

```kotlin
   @Provides
    @Singleton
    fun providerObjectDataBase(
        @ApplicationContext app: Context,
    ): AppDatabase = Room.databaseBuilder(
        app,
        ObjectDatabase::class.java,
        "NAME_DATABASE"
    ).build()
```


### Dependencys


```kotlin

// * id plugin
plugins {

    id 'kotlin-kapt'
    
}

dependencies {

// * room
    def roomVersion="2.4.3"
    implementation "androidx.room:room-runtime:$roomVersion"
    kapt "androidx.room:room-compiler:$roomVersion"
    implementation "androidx.room:room-ktx:$roomVersion"
    testImplementation "androidx.room:room-testing:$roomVersion"

}

```


## Hilt

### Application

```kotlin

@HiltAndroidApp
class Application : Application() {

    override fun onCreate() {
        super.onCreate()
        val formatStrategy: FormatStrategy = PrettyFormatStrategy.newBuilder()
            .showThreadInfo(true) // (Optional) Whether to show thread info or not. Default true
            .methodCount(1) // (Optional) How many method line to show. Default 2
            .methodOffset(5) // Set methodOffset to 5 in order to hide internal method calls
            .tag("") // To replace the default PRETTY_LOGGER tag with a dash (-).
            .build()

        Logger.addLogAdapter(AndroidLogAdapter(formatStrategy))


        Timber.plant(object : Timber.DebugTree() {

            override fun log(
                priority: Int, tag: String?, message: String, t: Throwable?,
            ) {
                Logger.log(priority, "@@", message, t)
            }
        })
    }
}

```

### Manifest

```
     <application
       ....
        android:name=".inject.Application"
```

### Basic Module

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object SimpleModule {

    @Provides
    @Singleton
    fun provideDependency(): RepositoryImpl =
        RepositoryImpl()
}
```

### RepositoryModule

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    @Singleton
    abstract fun provideObjectRepository(
        repositoryImpl: RepositoryImpl
    ):Repository
```

### Dependencys

```kotlin

plugins {

    id 'dagger.hilt.android.plugin'
    id 'kotlin-kapt'
    
}


dependencies {

    // * timber
    implementation 'com.orhanobut:logger:2.2.0'
    implementation 'com.jakewharton.timber:timber:4.7.1'


    // * hilt
    def daggerHiltVersion = '2.42'
    implementation "com.google.dagger:hilt-android:$daggerHiltVersion"
    kapt "com.google.dagger:hilt-compiler:$daggerHiltVersion"
    implementation 'androidx.hilt:hilt-navigation-compose:1.0.0'
    // ? hilt test
    testImplementation "com.google.dagger:hilt-android-testing:$daggerHiltVersion"
    androidTestImplementation "com.google.dagger:hilt-android-testing:$daggerHiltVersion"
    kaptAndroidTest "com.google.dagger:hilt-android-compiler:$daggerHiltVersion"
 }

```
#### id plugin search

```kotlin
 resolutionStrategy {
        eachPlugin {
            if (requested.id.id == 'dagger.hilt.android.plugin') {
                useModule("com.google.dagger:hilt-android-gradle-plugin:2.39.1")
            }
        }
    }

```

## Retrofit

```kotlin

#### Main interfaces

interface ObjectApiServices {


    @GET(OBJECT_PATH)
    suspend fun requestAllObjects(): ObjectApiResponse
    
    @POST(SIGN_UP_PATH)
    suspend fun signUp(
        @Body data: SignUpDTO
    ): AuthResponse
    
      
    @GET(OBJECT_PATH)
    suspend fun requestObject(
        @Header(AUTH_HEADER) token: String,
    ): ObjectApiResponse

}

 ```
 
 
 #### Module Retrofit
 
 ```kotlin
@Module
@InstallIn(SingletonComponent::class)
object ObjectApiServiceModule {
    private const val NAME_BASE_URL = "BaseUrl"

    @Named(NAME_BASE_URL)
    @Provides
    fun provideBaseUrl(): String = BASE_URL_API

    @Provides
    @Singleton
    fun provideGson(): Gson = GsonBuilder().create()

    @Provides
    @Singleton
    fun provideRetrofit(
        @Named(NAME_BASE_URL) baseUrl: String,
        gson: Gson,
    ): Retrofit = Retrofit.Builder()
        .baseUrl(baseUrl)
        .addConverterFactory(GsonConverterFactory.create(gson))
        .build()

    @Singleton
    @Provides
    fun provideDogsApiServices(
        retrofit: Retrofit,
    ): ObjectApiServices = retrofit.create(ObjectApiServices::class.java)
}
 
 ```
 
 #### Dependencys
 
 
 ```kotlin
 
 dependencies{
 
     // * gson
    implementation 'com.google.code.gson:gson:2.8.9'

    // * Retrofit
    def retrofitVersion = "2.9.0"
    implementation "com.squareup.retrofit2:retrofit:$retrofitVersion"
    implementation "com.squareup.retrofit2:converter-gson:$retrofitVersion"
    implementation "com.squareup.okhttp3:logging-interceptor:4.9.1"
    implementation "com.squareup.retrofit2:converter-scalars:$retrofitVersion"
    // ? retrofit test
 
 }
 
 ```
