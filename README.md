# Room
Biblioteka bazodawnowa bedąca kolejną warstwą abstrakcji dla SQLite. Jest częścią pakietu Jetpack.

### Encja
Klasa danych oznaczona adnotacją `@Entity`. Służy jako deklaracja tabeli w bazie danych.


```
@Entity(tableName="nazwa_tabeli")
data class MojaEncja(
  @PrimaryKey(autoGenerate=true)
  var id: Long = 0L //kluczem musi zero lub null
  @ColumnInfo(name="nazwa_kolumny) // zmiana nazwy kolumny jest opcjonalna
  val x: Int = 1
)
```

### Deklaracja Bazy
Deklaracja bazy danych Room odbywa się poprzez stworzenie klasy abstrakcyjnej. Klasa ta zawiera companion object, który pełni rolę singletona bazy.

```
@Database(
  entities = [MojaEncja::class], // kolejne encje po przecinku
  version = 1, // wersję inkrementujemy przy każdej wprowadzonej zmianie
  exportSchema = false // generator archiwum schematów
)
abstract class MyDatabase: RoomDatabase()

  abstract val myDao: MyDao // można mieć kilka DAO w jednej bazie
  
  companion object {
    @Volatile // jest to adnotacja z Javy, która sprawia że zmienna będzie widoczna dla wszystkich wątków. Należy zawsze z niej korzystać.
    private var INSTANCE: MyDatabase? = null
    
    fun getInstance(context: Context): MyDatabase {
      synchronized(this) {
        var instance = INSTANCE
        
        if(instance == null ) {
          instance = Room.databaseBuilder(context.applicationContext, MyDatabase::class.java, "my_database")
            .fallbackToDestructiveMigration() // strategia migracji / przy zmianie schematu stara baza zostanie usunięta
            .build
            INSTANCE = instance 
        }
      return INSTANCE
    }
```
        
  

### DAO
Bezpośrednie interakcje (zapytania) z bazą danych powinny przebiegać poprzez specjalny interface nazywany DAO ( Data Access Interface ). Nie tworzymy implementacji DAO - robi to za nas Room.
Adnotacje dostępne w DAO: `@Insert` `@Delete` `@Update` `@Query`. Adnotacja `Query` wymaga napisania własnego zapytania SQL.

```
@Dao
interface MyDatabaseDao {
  
  @Insert
  fun insert(mojaEncja: MojaEncja, innaEncja: InnaEncja)
  
  @Update
  fun update(mojaEncja: MojaEncja, innaEncja: InnaEncja)
  
  @Query("SELECT * FROM nazwa_tabeli WHERE id = :key")
  fun get(key: Long): MojaEncja?
}
```
 
