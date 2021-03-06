.. index::
   single: Формы

Формы
=====

Работа с формами - одна из наиболее типичных и проблемных задач для web-разработчика.
Symfony2 включает компонент для работы с Формами, который облегчает работу с ними.
В этой главе вы узнаете, как создавать сложные формы с нуля, познакомитесь с
наиболее важными особенностями библиотеки форм.

.. note::

   Компонент для работы с формами - это независимая библиотека, которая может
   быть использована вне проектов Symfony2. Подробности ищите по ссылке
   `Symfony2 Form Component`_ на ГитХабе.

.. index::
   single: Формы; Создание простой формы

Создание простой формы
----------------------

Предположим, вы работаете над простым приложением - списком ToDo, которое
будет отображать некоторые "задачи". Поскольку вашим пользователям будет
необходимо создавать и редактировать задачи, вам потребуется создать форму.
Но, прежде чем начать, давайте создадим базовый класс ``Task``, который
представляет и хранит данные для одной задачи:

.. code-block:: php

    <?php
    // src/Acme/TaskBundle/Entity/Task.php
    namespace Acme\TaskBundle\Entity;

    class Task
    {
        protected $task;

        protected $dueDate;

        public function getTask()
        {
            return $this->task;
        }
        public function setTask($task)
        {
            $this->task = $task;
        }

        public function getDueDate()
        {
            return $this->dueDate;
        }
        public function setDueDate(\DateTime $dueDate = null)
        {
            $this->dueDate = $dueDate;
        }
    }

.. note::

   Если вы будете брать код примеров один в один, вам, прежде всего
   необходимо создать пакет ``AcmeTaskBundle``, выполнив следующую команду
   (и принимая все опции интерактивного генератора по умолчанию):

   .. code-block:: bash

        php app/console generate:bundle --namespace=Acme/TaskBundle

Этот класс представляет собой обычный PHP-объект и не имеет ничего общего с
Symfony или какой-либо другой библиотекой. Это PHP-объект, который выполняет
задачу непосредственно внутри *вашего* приложение (т.е. является представлением
задачи в вашем приложении). Конечно же, к концу этой главы вы будете иметь
возможность отправлять данные для экземпляра ``Task`` (посредством
HTML-формы), валидировать её данные и сохранять в базу данных.

.. index::
   single: Формы; Создание формы в контроллере

Создание формы
~~~~~~~~~~~~~~~~~

Теперь, когда вы создали класс ``Task``, следующим шагом будет создание и
отображение HTML-формы. В Symfony2 это выполняется посредством создания
объекта формы и отображения его в шаблоне. Теперь, выполним необходимые
действия в контроллере:

.. code-block:: php

    <?php
    // src/Acme/TaskBundle/Controller/DefaultController.php
    namespace Acme\TaskBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Acme\TaskBundle\Entity\Task;
    use Symfony\Component\HttpFoundation\Request;

    class DefaultController extends Controller
    {
        public function newAction(Request $request)
        {
            // создаём задачу и присваиваем ей некоторые начальные данные для примера
            $task = new Task();
            $task->setTask('Write a blog post');
            $task->setDueDate(new \DateTime('tomorrow'));

            $form = $this->createFormBuilder($task)
                ->add('task', 'text')
                ->add('dueDate', 'date')
                ->getForm();

            return $this->render('AcmeTaskBundle:Default:new.html.twig', array(
                'form' => $form->createView(),
            ));
        }
    }

.. tip::

   Этот пример показывает, как создать вашу форму непосредственно в коде
   вашего контроллера. Позднее, в секции :ref:`book-form-creating-form-classes`,
   вы также узнаете, как создавать формы в отдельных классах, что является
   более предпочтительным вариантом и сделает ваши формы доступными для
   повторного использования.

Создание формы требует совсем немного кода, так как объекты форм в Symfony2
создаются при помощи конструктора форм - "form builder". Цель конструктора
форм - облегчить насколько это возможно создание форм, выполняя всю тяжёлую
работу.

В этом примере вы добавили два поля в вашу форму - ``task`` и ``dueDate``,
соответствующие полям ``task`` и ``dueDate`` класса ``Task``. Вы также
указали каждому полю их типы (например ``text``, ``date``), которые в числе
прочих параметров, определяют - какой HTML таг будет отображен для этого
поля в форме.

Symfony2 включает много встроенных типов, которые будут обсуждаться
совсем скоро (см. :ref:`book-forms-type-reference`).

.. index::
  single: Формы; Отображение формы в шаблоне

Отображение формы
~~~~~~~~~~~~~~~~~~

Теперь, когда форма создана, следующим шагом будет её отображение. Отобразить
форму можно, передав специальный объект "form view" в ваш шаблон (обратите
внимание на конструкцию ``$form->createView()`` в контроллере выше) и
использовать ряд функций-помощников в шаблоне:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}

        <form action="{{ path('task_new') }}" method="post" {{ form_enctype(form) }}>
            {{ form_widget(form) }}

            <input type="submit" />
        </form>

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Default/new.html.php -->

        <form action="<?php echo $view['router']->generate('task_new') ?>" method="post" <?php echo $view['form']->enctype($form) ?> >
            <?php echo $view['form']->widget($form) ?>

            <input type="submit" />
        </form>

.. image:: /images/book/form-simple.png
    :align: center

.. note::

    В этом примере предполагается, что вы создали маршрут ``task_new``,
    который указывает на контроллер ``AcmeTaskBundle:Default:new``,
    который был создан ранее.

Вот и всё! Напечатав ``form_widget(form)``, каждое поле формы будет отображено,
так же как метки полей и ошибки (если они есть). Это очень просто, но не
очень гибко (пока что). На практике вам, скорее всего, захочется отобразить
каждое поле формы отдельно, чтобы иметь полный контроль над тем как форма
выглядит. Вы узнаете, как сделать это в секции ":ref:`form-rendering-template`".

Прежде чем двигаться дальше, обратите внимание на то, как было отображено поле
``task``, содержащее значение поля ``task`` объекта ``$task`` ("Write a blog post").
Это - первая задача форм: получить данные от объекта и перевести их в формат,
подходящий для их последующего отображения в HTML форме.

.. tip::

   Система форм достаточно умна, чтобы получить доступ к значению защищённого
   (protected) поля ``task`` через методы ``getTask()`` и ``setTask()`` класса
   ``Task``. Так как поле не публичное (public), оно должно иметь "геттер" и
   "сеттер" методы для того, чтобы компонент форм мог получить данные из
   этого поля и изменить их. Для булевых полей вы также можете использовать
   "is*" метод (например ``isPublished()``) вместо ``getPublished()``.

.. index::
  single: Формы; Обработка отправки форм

Обработка отправки форм
~~~~~~~~~~~~~~~~~~~~~~~

Второй обязанностью форм является перевод данных, отправленных пользователем,
в свойства объекта. Для того чтобы это произошло, отправленные данные должны
быть привязаны к форме. Добавьте в контроллер следующие строки:

.. code-block:: php

    <?php
    // ...

    public function newAction(Request $request)
    {
        // создаём новый объект $task (без данных по умолчанию)
        $task = new Task();

        $form = $this->createFormBuilder($task)
            ->add('task', 'text')
            ->add('dueDate', 'date')
            ->getForm();

        if ($request->getMethod() == 'POST') {
            $form->bindRequest($request);

            if ($form->isValid()) {
                // выполняем прочие действие, например, сохраняем задачу в базе данных

                return $this->redirect($this->generateUrl('task_success'));
            }
        }

        // ...
    }

Теперь, при отправке формы контроллер привязывает отправленные данные к
форме, которая присваивает эти данные полям ``task`` и ``dueDate`` объекта
``$task``. Эта задача выполняется методом ``bindRequest()``.

.. note::

    Как только вызывается метод ``bindRequest()``, отправленные данные
    тут же присваиваются соответствующему объекту формы. Это происходит вне
    зависимости от того, валидны ли эти данные или нет.

Этот контроллер следует типичному сценарию по обработке форм и имеет три возможных
пути:

#. При первичной загрузке страницы в браузер метод запроса будет ``GET``,
   форма лишь создаётся и отображается;

#. Когда пользователь отправляет форму (т.е. метод будет уже ``POST``) с
   неверными данными (вопросы валидации будут рассмотрены ниже, а пока просто
   предположим что данные не валидны), форма будет привязана к данным и
   отображена вместе со всеми ошибками валидации;

#. Когда пользователь отправляет форму с валидными данными, форма будет
   привязана к данным и у вас есть возможность для выполнения некоторых
   действий, используя объект ``$task`` (например сохранить его в базе
   данных) перед тем как перенаправить пользователя на другую страницу
   (например, "thank you" или "success").

.. note::

   Перенаправление пользователя после успешной отправки формы предотвращает
   повторную отправку этих же данных, если пользователь обновит страницу.

.. index::
   single: Формы; Валидация

Валидация форм
--------------

В предыдущей секции вы узнали, что форма может быть отправлена с валидными
или не валидными данными. В Symfony2 валидация применяется к объекту, лежащему
в основе формы (например, ``Task``). Другими словами, вопрос не в том, валидна
ли форма, а валиден ли объект ``$task``, после того как форма передала
ему отправленные данные. Выполнив метод ``$form->isValid()``, можно узнать
валидны ли данные объекта ``$task`` или же нет.

Валидация выполняется посредством добавление набора правил (называемых ограничениями)
к классу. Для того, чтобы увидеть валидацию в действии, добавьте ограничения для
валидации того, что поле ``task`` не может быть пусто, а поле ``dueDate`` не может
быть пусто и должно содержать объект ``\DateTime``.

.. configuration-block::

    .. code-block:: yaml

        # Acme/TaskBundle/Resources/config/validation.yml
        Acme\TaskBundle\Entity\Task:
            properties:
                task:
                    - NotBlank: ~
                dueDate:
                    - NotBlank: ~
                    - Type: \DateTime

    .. code-block:: php-annotations

        // Acme/TaskBundle/Entity/Task.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Task
        {
            /**
             * @Assert\NotBlank()
             */
            public $task;

            /**
             * @Assert\NotBlank()
             * @Assert\Type("\DateTime")
             */
            protected $dueDate;
        }

    .. code-block:: xml

        <!-- Acme/TaskBundle/Resources/config/validation.xml -->
        <class name="Acme\TaskBundle\Entity\Task">
            <property name="task">
                <constraint name="NotBlank" />
            </property>
            <property name="dueDate">
                <constraint name="NotBlank" />
                <constraint name="Type">
                    <value>\DateTime</value>
                </constraint>
            </property>
        </class>

    .. code-block:: php

        <?php
        // Acme/TaskBundle/Entity/Task.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\NotBlank;
        use Symfony\Component\Validator\Constraints\Type;

        class Task
        {
            // ...

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('task', new NotBlank());

                $metadata->addPropertyConstraint('dueDate', new NotBlank());
                $metadata->addPropertyConstraint('dueDate', new Type('\DateTime'));
            }
        }

Это всё! Если вы отправите форму с ошибочными значениями - вы увидите
что соответствующие ошибки будут отображены в форме.

.. _book-forms-html5-validation-disable:

.. sidebar:: HTML5 Валидация

   Начиная с HTML5, многие браузеры могут выполнять некоторые валидационные
   ограничения на стороне клиента, без отправки формы. Типичным ограничением
   является указание атрибута ``required`` для полей, которые будут
   обязательными. В браузерах, которые поддерживают HTML5, этот атрибут будет
   позволять отображать браузерное сообщение об ошибке, если пользователь
   попробует отправить форму с пустым соответствующим полем.

   Генерированные формы полностью поддерживают эту возможность, добавляя
   соответствующие HTML-атрибуты, которые активируют HTML5 клиентскую
   валидацию. Тем не менее, валидация на стороне клиента может
   быть отключена путём добавления атрибута ``novalidate`` к тагу
   ``form`` или ``formnovalidate`` к тагу ``submit``. Это бывает необходимо,
   когда вам нужно протестировать ваши серверные ограничения, но,
   к примеру, браузер не даёт отправить форму с пустыми полями.

Валидация - это важная функция в составе Symfony2, она описывается
в :doc:`отдельной главе</book/validation>`.

.. index::
   single: Формы; Валидационные группы

.. _book-forms-validation-groups:

Валидационные группы
~~~~~~~~~~~~~~~~~~~~

.. tip::

    Если вы не используете :ref:`валидационные группы <book-validation-validation-groups>`,
    вы можете пропустить эту секцию.

Если ваш объект использует возможности :ref:`валидационных групп <book-validation-validation-groups>`,
вам нужно указать, какие группы вы хотите использовать:

.. code-block:: php

    <?php
    // ...

    $form = $this->createFormBuilder($users, array(
        'validation_groups' => array('registration'),
    ))->add(...)
    ;

Если вы создаёте :ref:`классы форм<book-form-creating-form-classes>`
(хорошая практика), тогда вам нужно указать следующий код в метод
``getDefaultOptions()``:

.. code-block:: php

    <?php
    // ...

    public function getDefaultOptions(array $options)
    {
        return array(
            'validation_groups' => array('registration')
        );
    }

В обоих этих примерах, для валидации объекта, для которого создана форма, будет
использована *лишь* группа ``registration``.

Groups based on Submitted Data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 2.1
   The ability to specify a callback or Closure in ``validation_groups``
   is new to version 2.1

Если вам требуется дополнительная логика для определения валидационных групп,
например, на совновании данных, отправленных пользователем, вы можете установить
значением опции validation_groups в массив с callback или замыкание (``Closure``).

.. code-block:: php

    <?php
    public function getDefaultOptions(array $options)
    {
        return array(
            'validation_groups' => array('Acme\\AcmeBundle\\Entity\\Client', 'determineValidationGroups'),
        );
    }

Этот код вызовет статический метод ``determineValidationGroups()`` класса Client с текущей формой в качестве
аргумента, после того как данные будут привязаны (bind) к форме, но перед запуском процесса валидации.
Вы также можете определить логику в замыкании ``Closure``, например:

.. code-block:: php

    <?php
    public function getDefaultOptions(array $options)
    {
        return array(
            'validation_groups' => function(FormInterface $form) {
                $data = $form->getData();
                if (Entity\Client::TYPE_PERSON == $data->getType()) {
                    return array('person')
                } else {
                    return array('company');
                }
            },
        );
    }

.. index::
   single: Формы; Встроенные типы полей

.. _book-forms-type-reference:

Встроенные типы полей
---------------------

В состав Symfony входит большое число типов полей, которые покрывают все
типичные поля и типы данных, с которыми вы столкнётесь:

.. include:: /reference/forms/types/map.rst.inc

Вы также можете создавать свои собственные типы полей. Эта возможность
подробно описывается в статье книги рецептов ":doc:`/cookbook/form/create_custom_field_type`".

.. index::
   single: Формы; Опции полей форм

Опции полей форм
~~~~~~~~~~~~~~~~

Каждый тип поля имеет некоторое число опций, которые можно использовать для
их настройки. Например, поле ``dueDate`` сейчас отображает 3 селектбокса.
Тем не менее, :doc:`поле date</reference/forms/types/date>` можно настроить
таким образом, чтобы отображался один текстбокс (где пользователь сможет
ввести дату в виде строки):

.. code-block:: php

    <?php
    // ...
    ->add('dueDate', 'date', array('widget' => 'single_text'))

.. image:: /images/book/form-simple2.png
    :align: center

Все типы полей имеют много различных опций, которые могут быть для них
указаны. Многие из этих опций - специфичны для конкретного типа поля и
более подробную информацию о них можно получить из справочника.

.. sidebar:: Опция ``required``

    Наиболее типичной опцией является опция ``required``, которая может быть
    указана для любого поля. По умолчанию, опция ``required`` установлена в
    ``true``, что даёт возможность браузерам с поддержкой HTML5 использовать
    встроенную в них клиентскую валидацию, если поле остаётся пустым. Если вам
    этого не требуется, или же установите опцию ``required`` в ``false`` или
    же :ref:`отключите валидацию HTML5<book-forms-html5-validation-disable>`.

    Отметим также, что установка опции ``required`` в ``true`` **не влияет**
    на серверную валидацию. Другими словами, если пользователь отправляет
    пустое значение для этого поля (при помощи старого браузера или веб-сервиса)
    оно будет считаться валидным, если вы не используете ограничения ``NotBlank``
    или ``NotNull``.

    Таким образом, опция ``required`` - хороша, но серверную валидацию
    использовать *необходимо всегда*.

.. index::
   single: Формы; Автоматическое определение типов полей

.. _book-forms-field-guessing:

Автоматическое определение типов полей
--------------------------------------

Теперь, когда вы добавили данные для валидации в класс ``Task``, Symfony
теперь много знает о ваших полях. Если вы позволите, Symfony может
определять ("угадывать") тип вашего поля и устанавливать его автоматически.
В этом примере, Symfony может определить по правилам валидации, что
``task`` является полем типа ``text`` и ``dueDate`` - полем типа ``date``:

.. code-block:: php

    <?php
    public function newAction()
    {
        $task = new Task();

        $form = $this->createFormBuilder($task)
            ->add('task')
            ->add('dueDate', null, array('widget' => 'single_text'))
            ->getForm();
    }

Автоматическое определение активируется, когда вы опускаете второй аргумент
в методе ``add()`` (или если вы указываете null для него). Если вы передаёте
массив опций в качестве третьего аргумента (как в случае ``dueDate`` выше),
эти опции применяются к "угаданному" полю.

.. caution::

    Если ваша форма использует особую группу валидации, определитель
    типов полей будет учитывать *все* ограничения при определении
    типов полей (включая ограничения, которые не являются частью
    используемых валидационных групп).

.. index::
   single: Формы; Автоматическое определение типов полей

Автоматическое определение опций для полей
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В дополнение к определению "типа" поля, Symfony также может попытаться
определить значения опций для поля.

.. tip::

    Когда эти опции будут установлены, поле будет отображено с
    использованием особых HTML атрибутов, которые позволяют выполнять
    HTML5 валидацию на стороне клиента (например ``Assert\MaxLength``).
    И, поскольку вам нужно будет вручную добавлять правила валидации
    на стороне сервера, эти опции могут быть угаданы исходя из ограничений,
    которые вы будете использовать для неё.

* ``required``: Опция ``required`` может быть определена исходя из правил
  валидации (т.е. если поле ``NotBlank`` или ``NotNull``) или же на основании
  метаданных Doctrine (т.е. если поле ``nullable``). Это может быть очень удобно,
  так как правила клиентской валидации автоматически соответствуют правилам
  серверной валидации.

* ``min_length``: Если поле является одним из видов текстовых полей,
  опция ``min_length`` может быть угадана исходя из правил валидации (
  если используются ограничения ``MinLength`` или ``Min``) или же из
  метаданных Doctrine (основываясь на длине поля).

* ``max_length``: Аналогично ``min_length`` с той лишь разницей, что определяет
  максимальное значение длины поля.

.. note::

  Эти опции могут быть определены автоматически, *только* если вы используете
  автоопределение полей (не указываете или передаёте ``null`` в качестве второго
  аргумента в метод ``add()``).

Если вы хотите изменить значения, определённые автоматически, вы можете
перезаписать их, передавая требуемые опции в массив опций::

    ->add('task', null, array('min_length' => 4))


.. index::
   single: Формы; Отображение в шаблоне

.. _form-rendering-template:

Отображение формы в шаблоне
------------------------------

Ранее вы увидели, как форму целиком можно отобразить при помощи лишь одной
строки кода. Конечно же, на практике вам потребуется большая гибкость при
отображении:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}

        <form action="{{ path('task_new') }}" method="post" {{ form_enctype(form) }}>
            {{ form_errors(form) }}

            {{ form_row(form.task) }}
            {{ form_row(form.dueDate) }}

            {{ form_rest(form) }}

            <input type="submit" />
        </form>

    .. code-block:: html+php

        <!-- // src/Acme/TaskBundle/Resources/views/Default/newAction.html.php -->

        <form action="<?php echo $view['router']->generate('task_new') ?>" method="post" <?php echo $view['form']->enctype($form) ?>>
            <?php echo $view['form']->errors($form) ?>

            <?php echo $view['form']->row($form['task']) ?>
            <?php echo $view['form']->row($form['dueDate']) ?>

            <?php echo $view['form']->rest($form) ?>

            <input type="submit" />
        </form>

Давайте более подробно рассмотрим каждую часть:

* ``form_enctype(form)`` - если хоть одно поле формы является полем для загрузки
  файла, эта функция отобразит необходимый атрибут ``enctype="multipart/form-data"``;

* ``form_errors(form)`` - Отображает глобальные по отношению к форме целиком ошибки
  валидации (ошибки для полей отображаются после них);

* ``form_row(form.dueDate)`` - Отображает текстовую метку, ошибки и HTML-виджет
  для заданного поля (например для ``dueDate``) внутри ``div`` элемента
  (по умолчанию);

* ``form_rest(form)`` - Отображает все остальные поля, которые ещё не были
  отображены. Как правило хорошая идея расположить вызов этого хелпера внизу
  каждой формы (на случай если вы забыли вывести какое-либо поле или же
  не хотите вручную отображать скрытые поля). Этот хелпер также удобен для
  активации автоматической защиты от :ref:`CSRF атак<forms-csrf>`.

Основная часть работы сделана при помощи хелпера ``form_row``, который
отображает метку, ошибки и виджет для каждого поля внутри тага ``div``.
В секции :ref:`form-theming` вы узнаете, как можно настроить вывод ``form_row``
на различных уровнях.

.. tip::

    Вы можете получить доступ к данным вашей формы при помощи ``form.vars.value``:

    .. configuration-block::

        .. code-block:: jinja

            {{ form.vars.value.task }}

        .. code-block:: html+php

            <?php echo $view['form']->get('value')->getTask() ?>

.. index::
   single: Формы; Отображение каждого поля вручную

Отображение каждого поля вручную
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Хелпер ``form_row`` очень удобен, так как вы можете быстро отобразить
каждое поле вашей формы (и разметка, используемая для каждой строки
может быть настроена). Но, так как жизнь как правило сложнее, чем нам
хотелось бы, вы можете также отобразить каждое поле вручную. В конечном
итоге вы получите тоже самое что и с хелпером ``form_row``:

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_errors(form) }}

        <div>
            {{ form_label(form.task) }}
            {{ form_errors(form.task) }}
            {{ form_widget(form.task) }}
        </div>

        <div>
            {{ form_label(form.dueDate) }}
            {{ form_errors(form.dueDate) }}
            {{ form_widget(form.dueDate) }}
        </div>

        {{ form_rest(form) }}

    .. code-block:: html+php

        <?php echo $view['form']->errors($form) ?>

        <div>
            <?php echo $view['form']->label($form['task']) ?>
            <?php echo $view['form']->errors($form['task']) ?>
            <?php echo $view['form']->widget($form['task']) ?>
        </div>

        <div>
            <?php echo $view['form']->label($form['dueDate']) ?>
            <?php echo $view['form']->errors($form['dueDate']) ?>
            <?php echo $view['form']->widget($form['dueDate']) ?>
        </div>

        <?php echo $view['form']->rest($form) ?>

Если автоматически созданная метка для поля вам не нравится, вы можете указать её
явно:

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_label(form.task, 'Task Description') }}

    .. code-block:: html+php

        <?php echo $view['form']->label($form['task'], 'Task Description') ?>

Наконец, некоторые типы полей имеют дополнительные опции отображения,
которые можно указывать виджету. Эти опции документированы для каждого
такого типа, но общей для всех опцией является ``attr``, которая позволяет
вам модифицировать атрибуты элемента формы. Следующий пример добавит текстовому
полю CSS класс ``task_field``:

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_widget(form.task, { 'attr': {'class': 'task_field'} }) }}

    .. code-block:: html+php

        <?php echo $view['form']->widget($form['task'], array(
            'attr' => array('class' => 'task_field'),
        )) ?>

Справочник по функциям Twig
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Если вы используете Twig, полная справочная информация о функциях, используемых
для отображения форм, доступна в :doc:`справочнике</reference/forms/twig_reference>`.
Ознакомьтесь с этой информацией, для того чтобы узнать больше о доступных хелперах
и опциях, которые для них доступны.

.. index::
   single: Формы; Создание классов форм

.. _book-form-creating-form-classes:

Создание классов форм
---------------------

Как вы уже видели ранее, форма может быть создана и использована
непосредственно в контроллере. Тем не менее, лучшей практикой является
создание формы в отдельном PHP-классе, который может быть использован повторно
в любом месте вашего приложения. Создайте новый класс, который будет содержать
логику создания формы ``task``:

.. code-block:: php

    <?php
    // src/Acme/TaskBundle/Form/Type/TaskType.php

    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;

    class TaskType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('task');
            $builder->add('dueDate', null, array('widget' => 'single_text'));
        }

        public function getName()
        {
            return 'task';
        }
    }

Этот новый класс содержит все необходимые указания для создания формы задачи
(обратите внимание, что метод ``getName()`` должен возвращать уникальный идентификатор
для данной формы). Теперь, вы можете использовать этот класс для быстрого создания
объекта формы в контроллере:

.. code-block:: php

    <?php
    // src/Acme/TaskBundle/Controller/DefaultController.php

    // добавьте use для класса формы в начале файла контроллера
    use Acme\TaskBundle\Form\Type\TaskType;

    public function newAction()
    {
        $task = // ...
        $form = $this->createForm(new TaskType(), $task);

        // ...
    }

Размещение логики формы в отдельном классе означает, что теперь форма может
быть легко использована в другом месте приложения. Это наилучший способ для
создания форм, но выбор конечно же за вами.

.. _book-forms-data-class:

.. sidebar:: Настройка ``data_class`` для формы

    Каждая форма должна знать имя класса, который будет содержать данные
    для неё (например, ``Acme\TaskBundle\Entity\Task``). Как правило,
    эти данные определяются автоматически по объекту, который передаётся
    вторым параметром в метод ``createForm`` (т.е. ``$task``). Позднее,
    когда вы займётесь встраиванием форм, полагаться на автоопреление уже
    будет нельзя. Таким образом, хоть и не всегда необходимо, но всё же
    желательно явно указывать опцию ``data_class``, добавив следующие
    строки в класс формы:

    .. code-block:: php

        <?php
        public function getDefaultOptions(array $options)
        {
            return array(
                'data_class' => 'Acme\TaskBundle\Entity\Task',
            );
        }

.. index::
   pair: Формы; Doctrine

Формы и Doctrine
------------------

Цель любой формы - преобразование данных из объекта (в нашем случае ``Task``) в
HTML форму и наоборот - преобразование данных, отправленных пользователем, обратно
в объект. По существу, тема по сохранению объекта ``Task`` в базе данных
совершенно не относится теме, обсуждаемой в главе "Формы". Тем не менее, если вы
сконфигурировали класс ``Task`` для работы с Doctrine (т.е. вы добавили
:ref:`метаданные для отображения<book-doctrine-adding-mapping>` (mapping metadata) для него), его
сохранение после отправки формы можно выполнить в случае, если форма валидна:

.. code-block:: php

    <?php
    if ($form->isValid()) {
        $em = $this->getDoctrine()->getEntityManager();
        $em->persist($task);
        $em->flush();

        return $this->redirect($this->generateUrl('task_success'));
    }

Если, по каким-то причинам у вас нет изначального объекта ``$task``,
вы можете получить его из формы::

    $task = $form->getData();

Больше информации по работе с базами данных вы можете получить в главе
:doc:`Doctrine ORM</book/doctrine>`.

Самое главное, что требуется уяснить, когда форма и данные связываются,
данные тут же передаются в объект, лежащий в основе формы. Если вы хотите
сохранить эти данные, вам нужно просто сохранить объект (который уже содержит
отправленные данные).

.. index::
   single: Формы; Встраивание форм

Встроенные формы
----------------

Зачастую, когда вы хотите создать форму, вам требуется добавлять в неё
поля из различных объектов. Например, форма регистрации может содержать
данные, относящиеся к объекту ``User`` и к нескольким объектам ``Address``.
К счастью, с использованием компонента форм сделать это легко и естественно.

Встраивание одного объекта
~~~~~~~~~~~~~~~~~~~~~~~~~~

Предположим, что каждая задача ``Task`` соответствует некоторому объекту
``Category``. Начнём конечно же с создания класса ``Category``:

.. code-block:: php

    <?php
    // src/Acme/TaskBundle/Entity/Category.php
    namespace Acme\TaskBundle\Entity;

    use Symfony\Component\Validator\Constraints as Assert;

    class Category
    {
        /**
         * @Assert\NotBlank()
         */
        public $name;
    }

Затем создадим свойство ``category`` в классе ``Task``:

.. code-block:: php

    <?php
    // ...

    class Task
    {
        // ...

        /**
         * @Assert\Type(type="Acme\TaskBundle\Entity\Category")
         */
        protected $category;

        // ...

        public function getCategory()
        {
            return $this->category;
        }

        public function setCategory(Category $category = null)
        {
            $this->category = $category;
        }
    }

Теперь ваше приложение нужно подправить с учётом новых требований. Создайте
класс формы для изменения объекта ``Category``:

.. code-block:: php

    <?php
    // src/Acme/TaskBundle/Form/Type/CategoryType.php
    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;

    class CategoryType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('name');
        }

        public function getDefaultOptions(array $options)
        {
            return array(
                'data_class' => 'Acme\TaskBundle\Entity\Category',
            );
        }

        public function getName()
        {
            return 'category';
        }
    }

Конечно целью же является изменение ``Category`` для ``Task`` непосредственно
из задачи. Для того чтобы выполнить это, добавьте поле ``category`` в
форму ``TaskType``, которое будет представлено экземпляром нового класса
``CategoryType``:

.. code-block:: php

    <?php
    public function buildForm(FormBuilder $builder, array $options)
    {
        // ...

        $builder->add('category', new CategoryType());
    }

Поля формы ``CategoryType`` теперь могут быть отображены прямо в форме ``TaskType``.
Отобразите поля ``Category`` тем же способом как и поля ``Task``:

.. configuration-block::

    .. code-block:: html+jinja

        {# ... #}

        <h3>Category</h3>
        <div class="category">
            {{ form_row(form.category.name) }}
        </div>

        {{ form_rest(form) }}
        {# ... #}

    .. code-block:: html+php

        <!-- ... -->

        <h3>Category</h3>
        <div class="category">
            <?php echo $view['form']->row($form['category']['name']) ?>
        </div>

        <?php echo $view['form']->rest($form) ?>
        <!-- ... -->

Когда пользователь отправляет форму, данные для полей ``Category`` будут
использованы для создания экземпляра ``Category``, который будет присвоен
полю ``category`` объекта ``Task``.

Объект ``Category`` доступен через метод ``$task->getCategory()`` и может
быть сохранён в базу данных или использован где требуется.

Встраивание коллекций форм
~~~~~~~~~~~~~~~~~~~~~~~~~~

Вы также можете встроить в вашу форму целую коллекцию форм (например форма ``Category``
с множеством саб-форм ``Product``). Этого можно достичь при использовании поля ``collection``.

Подробнее этот тип поля описан в книге рецептов ":doc:`/cookbook/form/form_collections`" и
справочнике: :doc:`collection</reference/forms/types/collection>`.

.. index::
   single: Формы; Дизайн
   single: Формы; Кастомизация полей

.. _form-theming:

Дизайн форм
-----------

Каждая часть отображения формы может быть настроена в соответствии с вашими
требованиями. Вы можете изменить как отображается каждая строка формы, изменить
разметку отображения ошибок и даже настроить как должен отображаться таг
``textarea``. Вы ничем не ограничены и разные настройки могут быть использованы
в разных местах.

Symfony использует шаблоны для отображения всех частей форм, таких как
метки, таги, input таги, сообщения об ошибках и многое другое.

В Twig каждый такой фрагмент представлен блоком Twig. Для настройки любой части
отображения формы вам просто надо заменить нужный блок.

В PHP каждый фрагмент формы отображается посредством индивидуального
файла шаблона. Для настройки отображения любой части формы вам нужно
заменить существующий шаблон новым.

Для того чтобы понять, как это работает, давайте настроим отображение
фрагмента ``form_row`` и добавим атрибут class для элемента ``div``,
который содержит каждую строку. Для того чтобы выполнить это, создайте новый файл
шаблона, который будет содержать новую разметку:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Form/fields.html.twig #}

        {% block field_row %}
        {% spaceless %}
            <div class="form_row">
                {{ form_label(form) }}
                {{ form_errors(form) }}
                {{ form_widget(form) }}
            </div>
        {% endspaceless %}
        {% endblock field_row %}

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Form/field_row.html.php -->

        <div class="form_row">
            <?php echo $view['form']->label($form, $label) ?>
            <?php echo $view['form']->errors($form) ?>
            <?php echo $view['form']->widget($form, $parameters) ?>
        </div>

Фрагмент ``field_row`` используется при отображении большинства полей при
помощи функции ``form_row``. Для того, чтобы сообщить компоненту форм, чтобы
он использовал новый фрагмент ``field_row``, определённый выше, добавьте
следующую строку в начале шаблона, отображающего форму:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}

        {% form_theme form 'AcmeTaskBundle:Form:fields.html.twig' %}

        <form ...>

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Default/new.html.php -->

        <?php $view['form']->setTheme($form, array('AcmeTaskBundle:Form')) ?>

        <form ...>

Таг ``form_theme`` (в Twig) как бы "импортирует" фрагменты, определённые в
указанном шаблоне и использует их при отображении формы. Другими словами,
когда вызывается функция ``form_row`` ниже в этом шаблоне, она будет
использовать блок ``field_row`` из вашей темы (вместо блока ``field_row``
по умолчанию используемого в Symfony).

Для того чтобы настроить любую часть формы, вам всего лишь нужно переопределить
все необходимые фрагменты. О том, какие блоки или файлы могут быть переопределены,
мы поговорим в следующей секции.

Дополнительную информацию о кастомизации форм ищите в книге рецептов:
:doc:`/cookbook/form/form_customization`.

.. index::
   single: Формы; Именование фрагментов форм

.. _form-template-blocks:

Именование фрагментов форм
~~~~~~~~~~~~~~~~~~~~~~~~~~

В Symfony, каждая отображаемая часть формы - HTML элементы форм, ошибки,
метки и т.д. - определены в базовой теме, которая представляет из себя
набор блоков в Twig и набор шаблонов в PHP.

В Twig все блоки определены в одном файле (`form_div_layout.html.twig`_),
который располагается внутри `Twig Bridge`_. В этом файле вы можете
увидеть любой из блоков, необходимых для отображения любого стандартного
поля.

В PHP каждый фрагмент расположен в отдельном файле. По умолчанию, они располагаются
в директории `Resources/views/Form` в составе пакета framework (`см. на GitHub`_).

Наименование каждого фрагмента следует одному базовому правилу и разбито на
две части, разделённых подчерком (``_``). Несколько примеров:

* ``field_row`` - используется функцией ``form_row`` для отображения большинства полей;
* ``textarea_widget`` - используется функцией ``form_widget`` для отображения полей типа ``textarea``;
* ``field_errors`` - используется функцией ``form_errors`` для отображения ошибок.

Каждый фрагмент подчиняется простому правилу: ``type_part``. Часть ``type`` соответствует
типу поля, которое будет отображено (например, ``textarea``, ``checkbox``, ``date`` и т.д.),
часть ``part`` соответствует же тому, что именно будет отображаться
(``label``, ``widget``, ``errors``, и т.д.). По умолчанию есть четыре возможных типов
*parts*, которые отображаются:

+-------------+--------------------------+-------------------------------------------------------------+
| ``label``   | (``field_label``)        | отображает метку для поля                                   |
+-------------+--------------------------+-------------------------------------------------------------+
| ``widget``  | (``field_widget``)       | отображает HTML-представление для поля                      |
+-------------+--------------------------+-------------------------------------------------------------+
| ``errors``  | (``field_errors``)       | отображает ошибки для поля                                  |
+-------------+--------------------------+-------------------------------------------------------------+
| ``row``     | (``field_row``)          | отображает цельную строку для поля (label, widget & errors) |
+-------------+--------------------------+-------------------------------------------------------------+

.. note::

    Есть также ещё три типа *parts* - ``rows``, ``rest``, и ``enctype`` -
    но заменять их вам вряд ли потребуется, так что и заботиться этом не стоит.

Зная тип поля (например ``textarea``), а также какую часть вы хотите изменить
(например, ``widget``), вы можете составить имя фрагмента, который должен быть
переопределён (например, ``textarea_widget``).

.. index::
   single: Формы; Наследование фрагментов шаблона

Наследование фрагментов шаблона форм
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В некоторых случаях фрагмент, который вам нужно настроить, не будет
существовать. Например, в базовой теме нет фрагмента ``textarea_errors``.
Но как же отображаются ошибки для полей textarea?

Ответ на этот вопрос такой: отображаются они при помощи фрагмента ``field_errors``.
Когда Symfony отображает ошибки для textarea, он ищет фрагмент
``textarea_errors``, прежде чем использовать стандартный фрагмент ``field_errors``.
Любой тип поля имеет *родительский* тип (для ``textarea`` это ``field``) и Symfony
использует фрагмент от родительского типа, если базовый фрагмент не существует.

Таким образом, чтобы переопределить фрагмент ошибок только для полей
``textarea``, скопируйте фрагмент ``field_errors``, переименуйте его
в ``textarea_errors`` и измените его как вам требуется. Для того, чтобы
изменить отображение ошибок для *всех* полей, скопируйте и измените
сам фрагмент ``field_errors``.

.. tip::

    Родительские типы для всех типов полей можно узнать из справочника:
    :doc:`типы полей</reference/forms/types>`.

.. index::
   single: Forms; Global Theming

Глобальная тема для форм
~~~~~~~~~~~~~~~~~~~~~~~~

В примере выше вы использовали хелпер ``form_theme`` (для Twig), чтобы
"импортировать" изменённые фрагменты форм *только* в одну форму. Вы также
можете указать Symfony тему форм для всего проекта в целом.

Twig
....

Для того, чтобы автоматически подключить переопределённые блоки из ранее
созданного шаблона ``fields.html.twig``, измените ваш файл конфигурации
следующим образом:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        twig:
            form:
                resources:
                    - 'AcmeTaskBundle:Form:fields.html.twig'
            # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->

        <twig:config ...>
                <twig:form>
                    <resource>AcmeTaskBundle:Form:fields.html.twig</resource>
                </twig:form>
                <!-- ... -->
        </twig:config>

    .. code-block:: php

        <?php
        // app/config/config.php

        $container->loadFromExtension('twig', array(
            'form' => array('resources' => array(
                'AcmeTaskBundle:Form:fields.html.twig',
             ))
            // ...
        ));

Любой блок внутри шаблона ``fields.html.twig`` будет использован глобально
в рамках проекта для определения формата отображения форм.

.. sidebar:: Настройка отображения форм в файле формы при использовании Twig

    При использовании Twig, вы также можете изменить блок формы непосредственно
    внутри шаблона, где требуется изменение стиля отображения:

    .. code-block:: html+jinja

        {% extends '::base.html.twig' %}

        {# import "_self" as the form theme #}
        {% form_theme form _self %}

        {# make the form fragment customization #}
        {% block field_row %}
            {# custom field row output #}
        {% endblock field_row %}

        {% block content %}
            {# ... #}

            {{ form_row(form.task) }}
        {% endblock %}

    Таг ``{% form_theme form _self %}`` позволяет изменять блоки формы
    непосредственно внутри того шаблона, который требует изменений. Используйте
    этот метод для быстрой настройки отображения формы, если данное
    изменение нигде больше не потребуется.

PHP
...

Для того, чтобы автоматически подключить изменённые шаблоны из директории
``Acme/TaskBundle/Resources/views/Form``, созданной ранее, для *всех* шаблонов,
измените конфигурацию вашего приложения следующим образом:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        framework:
            templating:
                form:
                    resources:
                        - 'AcmeTaskBundle:Form'
        # ...


    .. code-block:: xml

        <!-- app/config/config.xml -->

        <framework:config ...>
            <framework:templating>
                <framework:form>
                    <resource>AcmeTaskBundle:Form</resource>
                </framework:form>
            </framework:templating>
            <!-- ... -->
        </framework:config>

    .. code-block:: php

        <?php
        // app/config/config.php

        $container->loadFromExtension('framework', array(
            'templating' => array('form' =>
                array('resources' => array(
                    'AcmeTaskBundle:Form',
             )))
            // ...
        ));

Все фрагменты, определённые в директории ``Acme/TaskBundle/Resources/views/Form``
теперь будут использованы во всём приложении для изменения стиля отображения
форм.

.. index::
   single: Формы; Защита от CSRF атак

.. _forms-csrf:

Защита от CSRF атак
-------------------

CSRF - или же `Подделка межсайтовых запросов`_ это вид атак, позволяющий
злоумышленнику выполнять запросы от имени пользователей вашего приложения,
которые они делать не собирались (например перевод средств на счёт хакера).
К счастью, такие атаки можно предотвратить, используя CSRF токен в ваших формах.

Хорошие новости! Заключаются они в том, что Symfony по умолчанию добавляет
и валидирует CSRF токен для вас. Это означает, что вы получаете защиту от
CSRF атак не прилагая к этому никаких усилий. Фактически, все формы в этой
главе были защищены от подобных атак.

Защита от CSRF атак работает за счёт добавления в формы скрытого поля,
называемого по умолчанию ``_token``, которое содержит значение, которое
знаете только вы и пользователь вашего приложения. Это гарантирует, что
пользователь - и никто более - отправил данные, которые пришли к вам.
Symfony автоматически валидирует наличие и правильность этого токена.

Поле ``_token`` - это скрытое поле и оно автоматически отображается,
если вы используете функцию ``form_rest()`` в вашем шаблоне, которая
отображает все поля, которые ещё не были отображены в форме.

CSRF токен можно настроить уровне формы. Например:

.. code-block:: php

    <?php
    class TaskType extends AbstractType
    {
        // ...

        public function getDefaultOptions(array $options)
        {
            return array(
                'data_class'      => 'Acme\TaskBundle\Entity\Task',
                'csrf_protection' => true,
                'csrf_field_name' => '_token',
                // уникальный ключ для генерации секретного токена
                'intention'       => 'task_item',
            );
        }

        // ...
    }

Для того, чтобы отключить CSRF защиту, установите опцию ``csrf_protection`` в
``false``. Настройки также можно выполнить на уровне всего проекта. Дополнительную
информацию можно найти в :ref:`справочнике по настройке форм</reference-frameworkbundle-forms>`.

.. note::

    Опция ``intention`` (``намерение``) не обязательна, но значительно увеличивает
    безопасность сгенерированного токена, делая его различным для всех форм.

.. index:
   single: Формы; Без класса

Использование форм без класса
-----------------------------

В большинстве случаев форма привязывается к объекту и поля формы получают
и сохраняют данные в поля этого объекта. Это ровно то, с чем вы работали
ранее в этой главе.

Тем не менее, вам возможно потребуется использовать форму без соответствующего
класса и получать массив отправленных данных, а не объект. Этого просто достичь:

.. code-block:: php

    <?php
    // удостоверьтесь, что вы добавили use для пространства имён Request:
    use Symfony\Component\HttpFoundation\Request
    // ...

    public function contactAction(Request $request)
    {
        $defaultData = array('message' => 'Type your message here');
        $form = $this->createFormBuilder($defaultData)
            ->add('name', 'text')
            ->add('email', 'email')
            ->add('message', 'textarea')
            ->getForm();

            if ($request->getMethod() == 'POST') {
                $form->bindRequest($request);

                // data is an array with "name", "email", and "message" keys
                $data = $form->getData();
            }

        // ... render the form
    }

По умолчанию, форма полагает, что вы хотите работать с массивами данных, а
не с объектами. Есть два способа изменить это поведение и связать форму с
объектом:

1. Передать объект при создании формы (первый аргумент ``createFormBuilder``)
   или второй аргумент ``createForm``);

2. Определить опцию ``data_class`` для вашей формы.

Если вы этого *не сделали*, тогда форма будет возвращать данные в виде
массива. В этом примере, так как ``$defaultData`` не является объектом
(и не установлена опция ``data_class``), ``$form->getData()`` в конечном
итоге вернёт массив.

.. tip::

    Вы также можете получить доступ к значениям POST (в данном случае "name")
    напрямую через объект запроса, например так:

    .. code-block:: php

        $this->get('request')->request->get('name');

    Тем не менее, в большинстве случаев рекомендуется использовать метод
    getData(), так как он возвращает данные (как правило объект) после того
    как он был преобразован фреймворком форм.

Добавление валидации
~~~~~~~~~~~~~~~~~~~~

А как же быть с валидацией? Обычно, когда вы используете вызов ``$form->isValid()``,
объект валидировался на основании ограничений, которые вы добавили в этот класс.
Но когда класса нет, как добавить ограничения для данных из формы?

Ответом является настройка ограничений вручную и передача их в форму.
Полностью этот подход описан в главе о :ref:`Валидации<book-validation-raw-values>`,
мы же рассмотрим лишь небольшой пример:

.. code-block:: php

    <?php
    // импорт пространств имён
    use Symfony\Component\Validator\Constraints\Email;
    use Symfony\Component\Validator\Constraints\MinLength;
    use Symfony\Component\Validator\Constraints\Collection;

    $collectionConstraint = new Collection(array(
        'name' => new MinLength(5),
        'email' => new Email(array('message' => 'Invalid email address')),
    ));

    // создание формы без значений по умолчанию и с явным указанием ограничений для валидации
    $form = $this->createFormBuilder(null, array(
        'validation_constraint' => $collectionConstraint,
    ))->add('email', 'email')
        // ...
    ;

Теперь, когда вы вызываете `$form->isValid()`, ограничения, указанные выше,
выполняются для данных формы. Если вы используете класс формы, переопределите
метод ``getDefaultOptions``:

.. code-block:: php

    <?php
    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;
    use Symfony\Component\Validator\Constraints\Email;
    use Symfony\Component\Validator\Constraints\MinLength;
    use Symfony\Component\Validator\Constraints\Collection;

    class ContactType extends AbstractType
    {
        // ...

        public function getDefaultOptions(array $options)
        {
            $collectionConstraint = new Collection(array(
                'name' => new MinLength(5),
                'email' => new Email(array('message' => 'Invalid email address')),
            ));

            $options['validation_constraint'] = $collectionConstraint;
        }
    }

Теперь вы можете создавать формы с валидацией, которые возвращают массив
данных вместо объекта. В большинстве случаев же лучше - да и более правильно
- привязывать объекты к вашим формам. Но для простых форм вы можете этого и
не делать.

Заключение
--------------

Теперь вы знаете всё необходимое для создания сложных форм для вашего
приложения. При создании форм, не забывайте что первой целью формы
является транслирование данных из объекта (``Task``) в HTML форму, чтобы
пользователь мог модифицировать эти данные. Второй целью формы является
получение отправленных пользователем данных и передача их обратно в
объект.

Есть ещё много вещей, которые стоит узнать о прекрасном мире форм, таких
как :doc:`загрузка файлов при помощи Doctrine </cookbook/doctrine/file_uploads>`,
или же как создание формы с динамически меняемым числом вложенных форм (
например, список todo, где вы можете добавлять новые поля при помощи Javascript
перед отправкой). Ищите ответы в книге рецептов. Также изучите
:doc:`справочник по типам полей</reference/forms/types>`, который включает
примеры использования полей и их опций.

Читайте также в книге рецептов
------------------------------

* :doc:`/cookbook/doctrine/file_uploads`
* :doc:`Работа с полем File</reference/forms/types/file>`
* :doc:`Создание пользовательского поля </cookbook/form/create_custom_field_type>`
* :doc:`/cookbook/form/form_customization`
* :doc:`/cookbook/form/dynamic_form_generation`
* :doc:`/cookbook/form/data_transformers`

.. _`Symfony2 Form Component`: https://github.com/symfony/Form
.. _`DateTime`: http://php.net/manual/en/class.datetime.php
.. _`Twig Bridge`: https://github.com/symfony/symfony/tree/master/src/Symfony/Bridge/Twig
.. _`form_div_layout.html.twig`: https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_div_layout.html.twig
.. _`Подделка межсайтовых запросов`: http://ru.wikipedia.org/wiki/CSRF
.. _`см. на GitHub`: https://github.com/symfony/symfony/tree/master/src/Symfony/Bundle/FrameworkBundle/Resources/views/Form

.. toctree::
    :hidden:

    Translation source: 2011-10-02 8892b24
    Corrected from: 2011-10-16 2d0a37a
    Corrected from: 2011-12-06 53a7621

