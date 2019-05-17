# Дипломная работа к профессии Android-разработчик

## Хранение заметок

Есть множество подходов к тому как можно хранить наши заметки:

1. В памяти, но тогда после перезапуска приложения всё будет теряться;
2. В файлах, разделяя поля переносом строки;
3. В файлах, используя формат [json](https://ru.wikipedia.org/wiki/JSON);
4. В базе данных _sqlite_.

> Первый вариант не может быть использован в конечном приложении и не достаточен для получения зачета. Но он удобен на этапе разработки приложения.

Сортировка заметок:
1. Заметки сортируются по дате дедлайна: чем ближе срок истечения, тем выше заметка в списке (просроченные заметки оказываются в самом верху). 
2. Если дедлайны совпали или заметка не имеет дедлайна, тогда сортировка происходит по дате последнего изменения (новые или отредактированные оказываются выше).
3. Любая заметка с дедлайном всегда выше заметки без дедлайна.

Также как и в случае с хранилищем пин-кода работа с этим классом должна происходить через интерфейс:

```java
interface NoteRepository {
    Note getNoteById(String id);
    List<Note> getNotes();
    void saveNote(Note note);
    void deleteById(String id);
}
```
В таком случае в будущем мы сможем заменить одну реализацию другой. Смотри [внедрение зависимостей](app.md).

**Реализация**

! Это лишь одно из возможных решений. Рекомендую читать его после того, как вы попробовали реализовать это. !

Про класс Note:
id – должен быть уникальным. В зависимости от реализации алгоритм его получения может меняться, поэтому генерацией обязан заниматься сам репозиторий. Например для хранения в памяти подойдет любая случайная строка. Для хранения в виде файла накладываются ограничения, что строка не должна содержать недопустимых для имени файла символов. Для хранения в базе данных – _sqlite_ генерирует уникальные id самостоятельно.

Первая реализация, хранение в памяти:
Любая коллекция, которая окажется вам по душе, например, HashMap. Необходимую сортировку можно осуществлять с помощью `Arrays.sort()` и необходимого `Comparator`.

Реализация в виде файлов:
В данном случае мы будем сохранять заметку в файл по её id. Для того чтобы сохранить содержимое, будем располагать его на разных строчках: первая строка файла — заголовок, вторая строка — дата дедлайна, остальное содержимое файла – тело заметки. Сортировка та же, как и в случае с хранением в памяти. В данном случае в качестве даты последнего изменения можно использоваться дату последнего изменения файла `file.lastModified()`.

Реализация в виде файлов + _json_:
Здесь всё также как с простыми файлами, только нам не придется выдумывать структуру файла, мы будем сохранять строку _json_. Для получения из объекта в памяти его _json_ представления удобно использовать библиотеку [gson](https://github.com/google/gson). Но также можно воспользоваться стандартными средствами Android – [JSONObject](https://developer.android.com/reference/org/json/JSONObject).

Хранение в базе данных:
Для работы с базой данных _sqlite_ на Android, Google рекомендует использовать библиотеку [Room](https://developer.android.com/topic/libraries/architecture/room). Сортировка в данном случае должна быть реализована средствами базы данных, а не кодом на java. Также обратите внимание на то, что id генерируются самой базой данных.