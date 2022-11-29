# tips_android_programing

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
