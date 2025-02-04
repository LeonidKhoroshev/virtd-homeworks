
# Домашнее задание к занятию 1.  «Введение в виртуализацию. Типы и функции гипервизоров. Обзор рынка вендоров и областей применения» - Леонид Хорошев


## Задача 1. Опишите кратко, в чём основное отличие полной (аппаратной) виртуализации, паравиртуализации и виртуализации на основе ОС.

При аппаратной виртуализации гостевая ОС полностью изолирована виртуальной машиной от уровня виртуализации и аппаратного обеспечения, гипервизор может работать на уровне железа без установки ОС. (пример - Parallels Desktop).

Паравиртуализация использует hypercalls (привилегированные команды, которые позволяют виртуальной машине взаимодействовать с гипервизором) для операций обработки инструкций во время компиляции. При паравиртуализации гостевая ОС не полностью изолирована, но она частично изолирована виртуальной машиной от уровня виртуализации и аппаратного обеспечения (примеры - VMware, Xen, Hyper-V, Virtualbox). При паравиртуализации гостевая ОС знает, что она виртуализируется. При полной виртуализации  ОС работает так же, как если бы она не была виртуализирована, а вызовы ОС перехватываются и транслируются с использованием двоичного преобразования. Дополнительно по данному вопросу нашел отличную сравнительную [таблицу](https://www.geeksforgeeks.org/difference-between-full-virtualization-and-paravirtualization/).

Виртуализация на уровне ОС (контейнеризация) представляет собой выдедление изолированного пространства без эмуляции оборудования (процессор, память и т.д.). В ходе обучения и выполнения домашних заданий очень часто подобные контейнеры создавались внутри виртуальных машин, развернутых в Virtualbox (то есть средствами паравиртуализации). В поднятых контейнерах мы получаем отдельную корневую файловую систему, дерево процессов, сетевую подсистему, и так далее, но наша "изолированная среда" продолжает работать, используя ядро операционной системы, в которой она развернута (примеры - Docker, Cri-O). Сравнительная [таблица](https://u.netology.ru/backend/uploads/lms/attachments/files/data/44757/1._Введение_в_виртуализацию.pdf) приведена прямо в материалах к занятию.



## Задача 2. Выберите один из вариантов использования организации физических серверов в зависимости от условий использования.

Организация серверов:

- физические сервера,
- паравиртуализация,
- виртуализация уровня ОС.

Условия использования:

- высоконагруженная база данных, чувствительная к отказу;
- различные web-приложения;
- Windows-системы для использования бухгалтерским отделом;
- системы, выполняющие высокопроизводительные расчёты на GPU.

Опишите, почему вы выбрали к каждому целевому использованию такую организацию.

|  №  | Условия использования                      |  Организация серверов                          | Примечание                                                                |   
|--- |---------------------------------------------|-------------------------------------------------|---------------------------------------------------------------------------|
| 1  |  Высоконагруженная база данных,чувствительная к отказу | Паравиртуализация | В рамках одного физического сервера паравиртуализация позволяет собрать отказоустойчивый крастер для базы данных из нескольких нод.|
| 2  |  Различные web-приложения | Виртуализация уровня ОС | Контейнеризация позволяет удобно управлять web-приложениями, оперативно их рестартовать в случае сбоя, кроме того такие системы гораздо проще разворачивать в контейнерах (например через один Docker-compose файл можно запустить несколько LAMP серверов), также управлять несколькими контейнерами (например через Docker Swarm или Kubernetes) гораздо удобнее, чем парком виртуальных машин.| 
| 3  |  Windows-системы для использования бухгалтерским отделом |Паравиртуализация | Исходя из условий считаем, что в отделе используются машины с типовой конфигурацией для всех работников. Паравиртуализация позволяет удобно разворачивать и настраивать рабочие места сотрудников с применением Vagrant, Terraform и Ansible.|
| 4  |  Системы, выполняющие высокопроизводительные расчёты на GPU |  Физические сервера| В данном случае основное преимущество физического сервера в возможности использования всей вычислительной мощности, так как ресурсы не расходуются на гостевую ОС, гипервизоры и прочее программное обеспечение, обеспечивающее виртуализацию.|


## Задача 3. Выберите подходящую систему управления виртуализацией для предложенного сценария. Детально опишите ваш выбор.

| № | Сценарий | Система управления виртуализацией |
|----------|----------|----------|
| 1    |100 виртуальных машин на базе Linux и Windows, общие задачи, нет особых требований. Преимущественно Windows based-инфраструктура, требуется реализация программных балансировщиков нагрузки, репликации данных и автоматизированного механизма создания резервных копий. | Для Windows based-инфраструктуры логично использовать Hyper-V. Гипервизор позволяет создать балансировщик нагрузки, а также настроить репликацию данный как встоенными средствами так и сторонним софтом. Единственный недостаток, Mycrosoft в скором времени прекращает поддержку Hyper-V  (информация из открытых источников). Как альтернативу также можно рассматривать ESXi, один гипервизор позволяет развернуть до 120 виртуальных машин, а популярность платформы позволяет использовать программное обеспечение от многих компаний (актуально при миграции части инфраструктуры с программных продуктов Microsoft, например на Linux). |
| 2   | Требуется наиболее производительное бесплатное open source-решение для виртуализации небольшой (20-30 серверов) инфраструктуры на базе Linux и Windows виртуальных машин.   | В данной ситуации отлично подходит Proxmox - один из наиболее популярных гипервизоров с открытым исходным кодом. Поддерживает кластеризацию и позволяет осуществлять резервное копирование и живую миграцию виртуальных машин. Альтернативой по функционалу может служить ESXi, но он - проприетарный.  |
| 3    | Необходимо бесплатное, максимально совместимое и производительное решение для виртуализации Windows-инфраструктуры.  | Как и в первом примере отлично подойдет Hyper-V, во первых -  Windows-инфраструктура, во вторых гипервизор имеет бесплатные версии с открытым исходным кодом. |
| 4    | Необходимо рабочее окружение для тестирования программного продукта на нескольких дистрибутивах Linux.   | В данном случае отлично подойдет механизм контейнеризации, самый распространенный из которых - Docker. Если предпологается оркестрация контейнеров через Kubernetes, то лучше использовать Cri-o или Conteinerd, так как последние версии Kubernetes не поддерживают docker.   |

В данном не рассматривались такие решения, как например VirtualBox и VMWare, так как они предназначены больше для работы на персональный компьютерах, нежели чем на серверном оборудовании. 

## Задача 4. Опишите возможные проблемы и недостатки гетерогенной среды виртуализации (использования нескольких систем управления виртуализацией одновременно) и что необходимо сделать для минимизации этих рисков и проблем. Если бы у вас был выбор, создавали бы вы гетерогенную среду или нет? Мотивируйте ваш ответ примерами.

Поскольку гетерогенная среда неоднородна (как следует из определения), то наиболее вероятная проблема - это сложность эксплуатации такой инфраструктуры, из этого следует возможное снижение надежности (в случае недостатков эксплуатации) и рост затрат на обслуживание по сравнению с it инфраструктурой аналогичного масштаба, но основанной на едином технологическом стеке.

К примеру, инфраструктура компании в 50 виртуальных машин, полностью развернутая на серверном оборудовании с ESXi при корректно настроенном реплицировании, автоматическом создании снапшотов, бэкапов и т.д. в штатной ситуации практически не требует вмешательства (создание новых рабочих мест для сотрудников, миграция данных и т.д. осуществляется либо автоматически, либо одним работников с одного веб-интерфейса). А используя программное обеспечение с открытым исходным кодом (гипервизор Proxmox, ВМ под управлением Ubuntu и т.д.) затраты на инфраструктуру снижаются еще существеннее.

Так что, с точки зрения системного администратора, создания гетерогенной среды виртуализации следует избегать, однако, существуют случаи, когда такие решения необходимы.

Второй пример - аналогичная по размерам компания (50 виртуальных машин), но с более "сложной" сферой деятельности, например (условно) - электроэнергетика. 10 рабочих станций работает на под управлением ОС Windows (условная бухгалтерия запускает на них 1C и/или SAP) - для них необходим свой выделенный сервер, на котором в таких условиях целесообразней развернуть Hyper-V и уже в нем управлять рабочими местами административного персонала.

20 виртуальных машин работают под управлением ESXi, на них установлен Linux (к примеру Astra) и программное обеспечение SCADA, специально разработанное для мониторинга работы электросетевого оборудования. Все машины размещены на стороннем облачном сервисе (Яндекс облако к примеру). Еще 20 машин используются для сбора и обработки информации с приборов учета электроэнергии, администрирования базы данных, формирования суточных графиков перетоков и т.д. Поскольку информация критически важная с точки зрения бизнеса, то вопрос о ее хранении (физический сервер в офисе/облако открыт и решается в индивидуальном порядке для кажндого из филиалов).

При такой it архитектуре гетерогенная среда виртуализации неизбежна и все что мы можем сделать - это ее максимально упростить:
- сократить колическтво серверного оборудования (по моему мнению современные средства безопасности позволяют полностью развернуть подобную инфраструктуру в одном из облачных провайдеров);
- максимально вредрить (где это возможно) автоматизацию процессов (создание рабочих станций - Terraform, установка ПО - Ansible);
- настройка автоматического резервного копирования критически важных данных.

Скорее всего затраты не выйдут на уровень из примера 1, но вышеописанные мероприятия позволят не потерять в надежности работы системы и сохранности данных.

