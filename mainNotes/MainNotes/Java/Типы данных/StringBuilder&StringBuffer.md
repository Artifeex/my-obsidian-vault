Нужны для того, чтобы решить проблему с конкатенацией множества строк. Проблема связана с тем, что из-за не мутабельности строк каждое сложение плодит новый объект строки.

Под капотом работает как динамический массив. Т.е. при вызове метода append байты строки добавляются в динамический массив, который хранится внутри этих классов.

Различия StringBuilder&StringBuffer в том, что StringBuffer потокобезопасный. Т.е. все его методы помечены synchronized.
