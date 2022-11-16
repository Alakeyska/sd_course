# Создание ROS Publisher с использованием rospy

Самое время познакомиться с возможностями написания узлов на языке Python. В качестве основы можно посмотреть офф страницу о [написании узлов](http://wiki.ros.org/rospy_tutorials/Tutorials/WritingPublisherSubscriber).

Также можно добавить к полезным ссылкам [API rospy](http://docs.ros.org/api/rospy/html/) и ссылку на общее описание [Subscribers and Publishers](http://wiki.ros.org/rospy/Overview/Publishers%20and%20Subscribers).

Сначала взглянем на код, который будем разбирать:
```python
#!/usr/bin/env python
import rospy
from std_msgs.msg import String

rospy.init_node('talker')
pub = rospy.Publisher('my_chat_topic', String, queue_size=10)
rate = rospy.Rate(1)

def start_talker():
    msg = String()
    while not rospy.is_shutdown():
        hello_str = "hi =) %s" % rospy.get_time()
        rospy.loginfo(hello_str)

        msg.data = hello_str
        pub.publish(msg)

        rate.sleep()

try:
    start_talker()
except (rospy.ROSInterruptException, KeyboardInterrupt):
    rospy.logerr('Exception catched')
```

А теперь пошел разбор. Для начала, импортируем основной модуль `rospy` и модуль сообщения типа `std_msgs/String`. 
```python
#!/usr/bin/env python
import rospy
# Our case:     std_msgs/String     -> from std_msgs.msg import String
# Example:      geometry_msgs/Pose  -> from geometry_msgs.msg import Pose
from std_msgs.msg import String
```
> Строка `#!/usr/bin/env python` необходима для запуска, она сообщает системе о том, что данный файл необходимо запускать через интерпретатор `python`. 

После этого необходимо зарегистрировать узел в системе ROS, а также зарегистрировать топик на публикацию с указанием имени, типа сообщения для топика и размера очереди.
Первый аргумент функции `init_node()` задает название, которое будет зарегистрировано в рабочей экосистеме ROS.
Очередь нужна для сохранения сообщений, если узел публикует сообщения часто, при этом низкоуровневая передача сообщений работает медленнее или с задержками. Принцип выбора размера очереди хорошо описан [здесь](http://wiki.ros.org/rospy/Overview/Publishers%20and%20Subscribers). При переполнении очереди отправляются наиболее актуальные данные.
```python
rospy.init_node('talker')
pub = rospy.Publisher('my_chat_topic', String, queue_size=10)
```

При регистрации узла функцией `init_node()` у конструктора есть флаг `anonymous` со значение по-умолчанию `False`. Так при включении этого флага
```python
rospy.init_node('talker', anonymous=True)
```
к указанному имени узла добавляется суффикс (получается, например, `talker_18231_12354`), который делает узел уникальным в системе (свойство __анонимности__).

У функции `rospy.Publisher()` есть флаг `latch` со значение по-умолчанию `False`. Его включение добавляет следующее поведение: при отправке сообщений в топик сохраняется последнее отправленное сообщение и когда кто-нибудь подписался на этот топик - он сразу получает последнее сообщение из этого топика, даже если отправка была раньше, чем узел подписался на топик. Помогает, если в топик один раз опубликовались данные, а другой узел подписался на этот топик намного позже.

После остается только создать объект `Rate`, который используется для выдерживания частоты выполнения кода. В конструктор передается значение частоты в Гц.
```python
rate = rospy.Rate(1) # 1 Hz
```

На этом подготовка и создание необходимых объектов для простейшего узла готовы и пора перейти к основной логике программы.

В API ROS есть функция, которая сообщает о том, что система ROS завершила работу, именно ей и воспользуемся в качестве условия выхода `rospy.is_shutdown()`. Далее определим функцию с основной логикой узла для дальнейшего запуска.
```python
def start_talker():
    # Создаем объект сообщения
    msg = String()
    # Бесконечный цикл, пока ROS система работает
    while not rospy.is_shutdown():
        # Сформируем сообщение, которое включает в себя время
        hello_str = "hi =) %s" % rospy.get_time()
        # Вывод в терминал информации (содержание сообщения)
        rospy.loginfo(hello_str)
        # Заполнение сообщения и публикация сообщения в топик
        msg.data = hello_str
        pub.publish(msg)
        # Сон в соответствии с выдерживаемой частотой
        rate.sleep()
```

После этого можем запустить функцию узла (в ней находится вся логика). При этом заворачиваем в конструкцию `try-catch`, чтобы обработать исключение (нажатием Стоп или Ctrl+C в терминале). 
```python
try:
    start_talker()
except (rospy.ROSInterruptException, KeyboardInterrupt):
    rospy.logerr('Exception catched')
```


##### > Задачка по самостоятельной интеграции скрипта в наш пакет:
- Внутри пакета создать папку `scripts` (Python файлы считаются скриптами), в ней создать файл talker.py и в нем разместить код узла.
- Далее дать права на выполение с помощью команды `chmod +x talker.py`. Необходимо, чтобы вы находились в одной папке со скриптом.
- Попробовать запустить в системе ROS созданный узел, для ранее созданного пакета команда будет следующей:  
`rosrun study_pkg talker.py`
- Прим.: так как мы не задавали флаг анонимности в функции `rospy.init_node('talker')` (по-умолчанию там стоит `anonymous=False`), явно присваивать имя узла не требуется, оно будет такое, как было задано в функции `rospy.init_node()`
- Прим.: при возникновении каких-либо ошибок, они будут выведены в терминале 

##### > Напишите узел, который публикует в топик четные числа (0, 2, 4, 6 и т.д.) с частотой 10 Гц. Имена скрипта, узла и топика выберите сами. Проверьте корректность частоты публикации.

## В результате
- Был создан узел, публикующий сообщения в топик типа строки. Рассмотрено основное API пакета rospy.