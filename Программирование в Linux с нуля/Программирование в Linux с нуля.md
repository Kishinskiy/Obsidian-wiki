**Версия:** 0.094   
Оригинал: [lindevel.ru/zlp/](http://www.lindevel.ru/zlp/)

**_Copyright (c) 2003-2006 Nikolay N. Ivanov.  
Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free Documentation License, Version 1.2 or any later version published by the Free Software Foundation;_**

**_Каждый имеет право воспроизводить, распространять и/или вносить изменения в настоящий Документ в соответствии с условиями GNU Free Documentation License, Версией 1.2 или любой более поздней версией, опубликованной Free Software Foundation;_**

## Оглавление

[Предисловие](#preface_html)  
  
[Глава 1. ВВЕДЕНИЕ](#001_html)  
    [1.1. Что нужно знать](#001_html_1)  
    [1.2. Условные обозначения](#001_html_2)  
    [1.3. Что нужно иметь](#001_html_3)  
    [1.4. Обратная связь](#001_html_4)  
  
[Глава 2. ПЕРВЫЙ БЛИН](#002_html)  
    [2.1. Hello World](#002_html_1)  
    [2.2. Мультифайловое программирование](#002_html_2)  
    [2.3. Автоматическая сборка](#002_html_3)  
    [2.4. Модель КИС](#002_html_4)  
  
[Глава 3. БИБЛИОТЕКИ](#003_html)  
    [3.1. Введение в библиотеки](#003_html_1)  
    [3.2. Пример статической библиотеки](#003_html_2)  
    [3.3. Пример совместно используемой библиотеки](#003_html_3)  
  
[Глава 4. ОКРУЖЕНИЕ](#004_html)  
    [4.1. Введение в окружение](#004_html_1)  
    [4.2. Массив environ](#004_html_2)  
    [4.3. Чтение окружения: getenv()](#004_html_3)   
    [4.4. Запись окружения: setenv()](#004_html_4)   
    [4.5. Сырая модификация окружения: putenv()](#004_html_5)  
    [4.6. Удаление переменной окружения: unsetenv()](#004_html_6)  
    [4.7. Очистка окружения: clearenv()](#004_html_7)  
  
[Глава 5. НИЗКОУРОВНЕВЫЙ ВВОД-ВЫВОД](#005_html)  
    [5.1. Обзор механизмов ввода-вывода в Linux](#005_html_1)  
    [5.2. Файловые дескрипторы](#005_html_2)  
    [5.3. Открытие файла: системный вызов open()](#005_html_3)  
    [5.4. Закрытие файла: системный вызов close()](#005_html_4)  
    [5.5. Чтение файла: системный вызов read()](#005_html_5)  
    [5.6. Запись в файл: системный вызов write()](#005_html_6)  
    [5.7. Произвольный доступ: системный вызов lseek()](#005_html_7)  
  
[Глава 6. МНОГОЗАДАЧНОСТЬ](#006_html)  
    [6.1. Основы многозадачности в Linux](#006_html_1)  
    [6.2. Использование getpid() и getppid()](#006_html_2)  
    [6.3. Порождение процесса](#006_html_3)  
    [6.4. Замена образа процесса](#006_html_4)  
  
[Приложение 1: GNU Free Documentation License](#appendix01_html)  
  
[Приложение 2: Флаги режима доступа к файлу](#appendix02_html)  
    [Таблица 1. Флаги общего режима](#appendix02_html_1)  
    [Таблица 2. Флаги расширенного режима](#appendix02_html_2)  
    [Таблица 3. Дополнительные флаги](#appendix02_html_3)  
    [Таблица 4. Флаги режима открытия файла](#appendix02_html_4)