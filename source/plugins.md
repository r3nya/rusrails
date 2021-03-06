Основы создания плагинов Rails
==============================

Плагин Rails  - это либо расширение, либо изменение основного фреймворка. Плагины представляют:

* способ для разработчиков делиться новыми идеями без затрагивания стабильного кода
* сегментную архитектуру, такую, что часть кода может быть исправлена или изменена по своему собственному графику
* решение для разработчиков ядра приложения, чтобы не включать каждую новую особенность в свой код

После прочтения этого руководства, вы узнаете:

* Как создать плагин с нуля.
* Как написать и запустить тесты для плагина.

Это руководство описывает, как создать плагин, движимый тестами (TDD), который будет:

* Расширять классы ядра Ruby, такие как Hash и String.
* Добавлять методы в ActiveRecord::Base в традициях плагинов 'acts_as'.
* Представлять информацию о том, где разместить генераторы в вашем плагине.

Для целей этого руководства представьте на момент, что вы заядлый любитель птиц. Вашей любимой птицей является дятел (Yaffle), и вы хотите создать плагин, позволяющий другим разработчикам пользоваться особенностями дятлов.

Настройка
---------

В настоящий момент плагины Rails создаются как гемы, _gem_. Они могут использоваться различными приложениями с помощью RubyGems и Bundler.

### Создание гема.

Rails поставляется с командой `rails plugin new`, создающей скелет для разработки любого типа расширения Rails со способностью запуска интеграционных тестов с помощью приложения-заглушки Rails. Как ее использовать и ее опции смотрите:

```bash
$ rails plugin --help
```

Тестирование своего нового плагина
----------------------------------

Можете перейти в директорию, содержащую плагин, запустить команду `bundle install`, и запустить сгенерированный тест с использованием команды `rake`.

Вы должны увидеть:

```bash
  2 tests, 2 assertions, 0 failures, 0 errors, 0 skips
```

Это сообщает вам, что все сгенерировалось правильно, и вы можете начать добавлять функционал.

Расширение классов ядра
-----------------------

Этот раздел объясняет, как добавить метод в String, который будет доступен везде в вашем приложении rails.

В следующем примере мы добавим метод в String с именем `to_squawk`. Для начала создайте новый файл теста с несколькими утверждениями:

```ruby
# yaffle/test/core_ext_test.rb

require 'test_helper'

class CoreExtTest < Test::Unit::TestCase
  def test_to_squawk_prepends_the_word_squawk
    assert_equal "squawk! Hello World", "Hello World".to_squawk
  end
end
```

Запустите `rake` для запуска теста. Этот тест должен провалиться, так как мы еще не реализовали метод `to_squawk`:

```bash
    1) Error:
  test_to_squawk_prepends_the_word_squawk(CoreExtTest):
  NoMethodError: undefined method `to_squawk' for [Hello World](String)
      test/core_ext_test.rb:5:in `test_to_squawk_prepends_the_word_squawk'
```

Отлично - теперь мы готовы начать разработку.

Затем в `lib/yaffle.rb` добавьте `require "yaffle/core_ext"`:

```ruby
# yaffle/lib/yaffle.rb

require "yaffle/core_ext"

module Yaffle
end
```

Наконец, создайте файл `core_ext.rb` и добавьте метод `to_squawk`:

```ruby
# yaffle/lib/yaffle/core_ext.rb

String.class_eval do
  def to_squawk
    "squawk! #{self}".strip
  end
end
```

Чтобы проверить, что этот метод делает то, что нужно, запустите юнит тесты с помощью `rake` из директории плагина.

```bash
  3 tests, 3 assertions, 0 failures, 0 errors, 0 skips
```

Чтобы увидеть его в действии, измените директорию на test/dummy, запустите консоль и начните squawking:

```bash
$ rails console
>> "Hello World".to_squawk
=> "squawk! Hello World"
```

Добавление метода "acts_as" в Active Record
----------------------------------------

Обычным паттерном для плагинов является добавление в модель метода с именем 'acts_as_something'. В нашем случае мы хотим написать метод с именем 'acts_as_yaffle', добавляющий метод 'squawk' в модель Active Record.

Для начала настройте свои файлы, вам нужны:

```ruby
# yaffle/test/acts_as_yaffle_test.rb

require 'test_helper'

class ActsAsYaffleTest < Test::Unit::TestCase
end
```

```ruby
# yaffle/lib/yaffle.rb

require "yaffle/core_ext"
require 'yaffle/acts_as_yaffle'

module Yaffle
end
```

```ruby
# yaffle/lib/yaffle/acts_as_yaffle.rb

module Yaffle
  module ActsAsYaffle
    # your code will go here
  end
end
```

### Добавление метода класса

Этот плагин ожидает, что мы добавим в модель метод с именем 'last_squawk'. Однако, у пользователей плагина уже может быть определен метод в модели 'last_squawk', который они используют для чего-то иного. Этот плагин позволит имени быть измененным, добавив метод класса 'yaffle_text_field'.

Для начала напишем падающий тест, показывающий нужное нам поведение:

```ruby
# yaffle/test/acts_as_yaffle_test.rb

require 'test_helper'

class ActsAsYaffleTest < Test::Unit::TestCase

  def test_a_hickwalls_yaffle_text_field_should_be_last_squawk
    assert_equal "last_squawk", Hickwall.yaffle_text_field
  end

  def test_a_wickwalls_yaffle_text_field_should_be_last_tweet
    assert_equal "last_tweet", Wickwall.yaffle_text_field
  end

end
```

При запуске `rake` вы увидите следующее:

```
    1) Error:
  test_a_hickwalls_yaffle_text_field_should_be_last_squawk(ActsAsYaffleTest):
  NameError: uninitialized constant ActsAsYaffleTest::Hickwall
      test/acts_as_yaffle_test.rb:6:in `test_a_hickwalls_yaffle_text_field_should_be_last_squawk'

    2) Error:
  test_a_wickwalls_yaffle_text_field_should_be_last_tweet(ActsAsYaffleTest):
  NameError: uninitialized constant ActsAsYaffleTest::Wickwall
      test/acts_as_yaffle_test.rb:10:in `test_a_wickwalls_yaffle_text_field_should_be_last_tweet'

  5 tests, 3 assertions, 0 failures, 2 errors, 0 skips
```

Это сообщает нам об отсутствии необходимых моделей (Hickwall и Wickwall), которые мы пытаемся протестировать. Эти модели можно с легкостью создать в нашем  "dummy" приложении Rails, запустив следующие команды в директории test/dummy:

```bash
$ cd test/dummy
$ rails generate model Hickwall last_squawk:string
$ rails generate model Wickwall last_squawk:string last_tweet:string
```

Теперь можно создать необходимые таблицы в вашей тестовой базе данных, перейдя в приложение-заглушку и мигрировав базу данных. Сначала

```bash
$ cd test/dummy
$ rake db:migrate
$ rake db:test:prepare
```

Пока вы тут, измените модели Hickwall и Wickwall так, чтобы они знали, что они должны действовать как дятлы.

```ruby
# test/dummy/app/models/hickwall.rb

class Hickwall < ActiveRecord::Base
  acts_as_yaffle
end

# test/dummy/app/models/wickwall.rb

class Wickwall < ActiveRecord::Base
  acts_as_yaffle yaffle_text_field: :last_tweet
end

```

Также добавим код, определяющий метод acts_as_yaffle.

```ruby
# yaffle/lib/yaffle/acts_as_yaffle.rb
module Yaffle
  module ActsAsYaffle
    extend ActiveSupport::Concern

    included do
    end

    module ClassMethods
      def acts_as_yaffle(options = {})
        # тут будет ваш код
      end
    end
  end
end

ActiveRecord::Base.send :include, Yaffle::ActsAsYaffle
```

Затем можно вернуться в корневую директорию плагина (`cd ../..`) и перезапустить тесты с помощью `rake`.

```
    1) Error:
  test_a_hickwalls_yaffle_text_field_should_be_last_squawk(ActsAsYaffleTest):
  NoMethodError: undefined method `yaffle_text_field' for #<Class:0x000001016661b8>
      /Users/xxx/.rvm/gems/ruby-1.9.2-p136@xxx/gems/activerecord-3.0.3/lib/active_record/base.rb:1008:in `method_missing'
      test/acts_as_yaffle_test.rb:5:in `test_a_hickwalls_yaffle_text_field_should_be_last_squawk'

    2) Error:
  test_a_wickwalls_yaffle_text_field_should_be_last_tweet(ActsAsYaffleTest):
  NoMethodError: undefined method `yaffle_text_field' for #<Class:0x00000101653748>
      Users/xxx/.rvm/gems/ruby-1.9.2-p136@xxx/gems/activerecord-3.0.3/lib/active_record/base.rb:1008:in `method_missing'
      test/acts_as_yaffle_test.rb:9:in `test_a_wickwalls_yaffle_text_field_should_be_last_tweet'

  5 tests, 3 assertions, 0 failures, 2 errors, 0 skips

```

Подбираемся ближе... Теперь мы реализуем код метода acts_as_yaffle, чтобы тесты проходили.

```ruby
# yaffle/lib/yaffle/acts_as_yaffle.rb

module Yaffle
  module ActsAsYaffle
   extend ActiveSupport::Concern

    included do
    end

    module ClassMethods
      def acts_as_yaffle(options = {})
        cattr_accessor :yaffle_text_field
        self.yaffle_text_field = (options[:yaffle_text_field] || :last_squawk).to_s
      end
    end
  end
end

ActiveRecord::Base.send :include, Yaffle::ActsAsYaffle
```

Когда запустите `rake`, все тесты должны пройти:

```bash
  5 tests, 5 assertions, 0 failures, 0 errors, 0 skips
```

### Добавление метода экземпляра

Этот плагин добавит метод 'squawk' в любой объект Active Record, который вызовет 'acts_as_yaffle'. Метод 'squawk' просто установит значение одному из полей в базе данных.

Для начала напишем падающий тест, показывающий желаемое поведение:

```ruby
# yaffle/test/acts_as_yaffle_test.rb
require 'test_helper'

class ActsAsYaffleTest < Test::Unit::TestCase

  def test_a_hickwalls_yaffle_text_field_should_be_last_squawk
    assert_equal "last_squawk", Hickwall.yaffle_text_field
  end

  def test_a_wickwalls_yaffle_text_field_should_be_last_tweet
    assert_equal "last_tweet", Wickwall.yaffle_text_field
  end

  def test_hickwalls_squawk_should_populate_last_squawk
    hickwall = Hickwall.new
    hickwall.squawk("Hello World")
    assert_equal "squawk! Hello World", hickwall.last_squawk
  end

  def test_wickwalls_squawk_should_populate_last_tweet
    wickwall = Wickwall.new
    wickwall.squawk("Hello World")
    assert_equal "squawk! Hello World", wickwall.last_tweet
  end
end
```

Запустите тест, чтобы убедиться, что последние два теста упадут с ошибкой, содержащей "NoMethodError: undefined method `squawk'", затем обновите 'acts_as_yaffle.rb', чтобы он выглядел так:

```ruby
# yaffle/lib/yaffle/acts_as_yaffle.rb

module Yaffle
  module ActsAsYaffle
    extend ActiveSupport::Concern

    included do
    end

    module ClassMethods
      def acts_as_yaffle(options = {})
        cattr_accessor :yaffle_text_field
        self.yaffle_text_field = (options[:yaffle_text_field] || :last_squawk).to_s

        include Yaffle::ActsAsYaffle::LocalInstanceMethods
      end
    end

    module LocalInstanceMethods
      def squawk(string)
        write_attribute(self.class.yaffle_text_field, string.to_squawk)
      end
    end
  end
end

ActiveRecord::Base.send :include, Yaffle::ActsAsYaffle
```

Запустите `rake` в последний раз, вы должны увидеть:

```
  7 tests, 7 assertions, 0 failures, 0 errors, 0 skips
```

NOTE: Использование `write_attribute` для записи в поле модели - это всего лишь пример того, как плагин может взаимодействовать с моделью, но не всегда правильный метод для использования. Например, также можно использовать `send("#{self.class.yaffle_text_field}=", string.to_squawk)`.

Генераторы
----------

Генераторы могут быть включены в ваш гем простым их добавлением в директорию lib/generators вашего плагина. Подробнее о создании генераторов смотрите в [руководстве по генераторам](/generators)

Публикация вашего гема
----------------------

Плагины в виде гемов, которые в текущий момент в разработке, могут с легкостью быть доступны из любого репозитория Git. Чтобы поделиться гемом Yaffle с другими, просто передайте код в репозиторий Git (такой как GitHub) и добавьте строчку в Gemfile требуемого приложения:

```ruby
gem 'yaffle', git: 'git://github.com/yaffle_watcher/yaffle.git'
```

После запуска `bundle install` функционал вашего гема будет доступен в приложении.

Когда гем будет готов стать доступным в виде формального релиза, он может быть опубликован на [RubyGems](http://www.rubygems.org). Подробнее о публикации гемов на RubyGems смотрите: [Creating and Publishing Your First Ruby Gem](http://blog.thepete.net/2010/11/creating-and-publishing-your-first-ruby.html)

Документация RDoc
-----------------

Как только ваш плагин станет стабильным, и вы будете готовы его разместить, сделайте хорошее дело, документировав его! К счастью, написание документации для вашего плагина - это очень просто.

Первым шагом является обновление файла README детальной информацией о том, как использовать ваш плагин. Ключевые вещи, которые следует включить, следующие:

* Ваше имя
* Как установить
* Как добавить функционал в приложение (несколько примеров обычных ситуаций использования)
* Предупреждения, хитрости или подсказки, которые могут помочь пользователям и сохранить их время

Как только README готов, пройдитесь и добавьте комментарии rdoc ко всем методам, которые будут использовать разработчики. Также принято добавить комментарии '#:nodoc:' к тем частям кода, которые не включены в публичный API.

Как только ваши комментарии закончены, перейдите в директорию плагины и запустите:

```bash
$ rake rdoc
```

### Ссылки

* [Developing a RubyGem using Bundler](https://github.com/radar/guides/blob/master/gem-development.md)
* [Using .gemspecs as Intended](http://yehudakatz.com/2010/04/02/using-gemspecs-as-intended/)
* [Gemspec Reference](http://docs.rubygems.org/read/chapter/20)
* [GemPlugins: A Brief Introduction to the Future of Rails Plugins](http://www.intridea.com/blog/2008/6/11/gemplugins-a-brief-introduction-to-the-future-of-rails-plugins)
