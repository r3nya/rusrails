# Ошибки конфигурации

Простейший способ работы с Rails заключается в хранении всех внешних данных в UTF-8. Если не так, библиотеки Ruby и Rails часто будут способны конвертировать ваши родные данные в UTF-8, но это не всегда надежно работает, поэтому лучше быть уверенным, что все внешние данные являются UTF-8.

Если вы допускаете ошибку в этой области, наиболее обычным симптомом является черный ромбик со знаком вопроса внутри, появляющийся в браузере. Другим обычным симптомом являются символы, такие как "Ã¼" появляющиеся вместо "ü". Rails предпринимает ряд внутренних шагов для смягчения общих случаев тех проблем, которые могут быть автоматически обнаружены и исправлены. Однако, если имеются внешние данные, не хранящиеся в UTF-8, это может привести к такого рода проблемам, которые не могут быть автоматически обнаружены Rails и исправлены.

Два наиболее обычных источника данных, которые не в UTF-8:

* Ваш текстовый редактор: Большинство текстовых редакторов (такие как Textmate), по умолчанию сохраняют файлы как UTF-8. Если ваш текстовый редактор так не делает, это может привести к тому, что специальные символы, введенные в ваши шаблоны (такие как é) появятся как ромбик с вопросительным знаком в браузере. Это также касается ваших файлов перевода i18N. Большинство редакторов, не устанавливающие по умолчанию UTF-8 (такие как некоторые версии Dreamweaver) предлагают способ изменить умолчания на UTF-8. Сделайте так.
* Ваша база данных. Rails по умолчанию преобразует данные из вашей базы данных в UTF-8 на границе. Однако, если ваша база данных не использует внутри UTF-8, она может не быть способной хранить все символы, которые введет ваш пользователь. Например, если ваша база данных внутри использует Latin-1, и ваш пользователь вводит русские, ивритские или японские символы, данные будут потеряны как только попадут в базу данных. Если возможно, используйте UTF-8 как внутреннее хранилище в своей базе данных.