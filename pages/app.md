# Дипломная работа к профессии Android-разработчик

## Внедрение зависимостей

*Эта часть обязательна для получения зачета.*

Что это?
Сухим языком это инверсия управления, когда не сам класс решает каким именно образом будет что-то делать, а получает реализацию из вне.

Зачем это нам?
Мы с этим столкнулись в `Keystore` и `NoteRepository`. Там оказалось что мы можем сделать хранение пин-кода и заметок разными способами, за которые будут отвечать разные классы. Но с другой стороны, тем кто эти классы использует, например `ListNotesActivity` – совершенно не интересно как именно это происходит, хранятся ли заметки в файлах или в памяти.

Как реализовать подмену в нашем приложении?
Для того чтобы снять ответственность за выбор реализации (память, файлы, sqlite) с классов которые этим не должны заниматься, нужно перенести логику создания конкретной реализации в другое место, а `ListNotesActivity` будет получать готовую реализацию репозиторий с заметками.

*class App*:
В Android в качестве такого места удобно использовать класс App, который представляет собой экземпляр приложения, для этого нам нужно:
1. Унаследовать свой класс от `android.app.Application`;
2. Создать статические переменные, хранящие необходимые нам реализации интерфейсов и геттеры для них;
3. Переопределить метод `onCreate()` и в нём создать необходимые нам реализации классов. Обратите внимание класс `Application` является наследником `Context`, так что если нам нужен `Context` мы можем использовать `this`;
4. В приложении, если нам понадобился `NoteRepository` или `Keystore` мы можем обратится к `App.getNoteRepository()` и получить реализацию, причем нам не важно какую именно!

```java
public class App extends Application {
    private static NoteRepository noteRepository;
    private static Keystore keystore;
    @Override
    public void onCreate() {
        super.onCreate();

        /* Конкретная реализация выбирается только здесь.
           Изменением одной строчки здесь, 
           мы заменяем реализацию во всем приложении!
        */

        noteRepository = new FileNoteRepository(this);
        passwordStorage = new SimpleKeystore(this);
    }
    // Возвращаем интерфейс, а не конкретную реализацию!
    public static NoteRepository getNoteRepository() {
        return noteRepository;
    }
    // Возвращаем интерфейс, а не конкретную реализацию!
    public static Keystore getKeystore() {
        return keystore;
    }
}
```