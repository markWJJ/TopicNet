# Cubes
[English version](README.md)
Виды кубиков (cubes):

* `BaseCube` — абстрактный класс для создания других кубиков
* `RegularizersModifierCube` — кубик добавляющий/изменяющий регуляризаторы
* `CubeCreator` - куб для перебора начальных параметров модели.
* `ClassIdModifierCube` - куб для перебора class_ids
---

## Внутреннее устройство кубика

**Основные атрибуты кубика**:

* `parameters` — основной атрибут кубика.
Содержит информацию о параметрах всех возможных изменений со всеми возможными коэффициентами.
Может являться любым `iterable` объектом.
Важно, чтобы один этап (одно множество гиперпараметров) применения кубика определялся одним элементом `parameters`.

    *Например, для кубика, в котором для регуляризатора SmoothSparsePhi перебираются 
    пять коэффициентов регуляризации, атрибут `parameters`
    должен содержать пять элементов,
    каждый из которых должен как миниму содержать тип регуляризатора
    и текущее значение коэффициента регуляризации.*

**Основные методы кубика**:

* `__init__` — в методе происходит инициализация параметров кубика,
в частности в `__init__` должен задаваться атрибут `parameters`.

* `__call__` — применение кубика к модели на выбранном наборе данных.
Вход функции — инстанс класса TopicModel и инстанс Dataset, задающий данные для обучнеия.
Основное использование кубика происходит через этот метод. 
Внутри `__call__` происходит логирование процесса обучения,
перебор возможных моделей по заданной стратегии
(например, при помощи метода `apply` для каждого набора параметров может создаваться копия старой модели, после чего 
параметры копии изменяются заданным образом),
обучение всех моделей заданное число итераций и обновление сущностей, связанных с кубиком.

* `apply` — создание модели или изменение параметров модели согласно заданному набору параметров.
Функция возвращает модель.
Эта функция специфична для каждого кубика. 

    *Например, для кубика, в котором для регуляризатора SmoothSparsePhi
    перебираются пять коэффициентов регуляризации,
    данная функция будет копировать исходную модель и добавлять новый регуляризатор с нужным коэффициентом
    регуляризации.* 

* `get_jsonable_from_parameters` — преобразование атрибута `parameters` в структуру, которую можно 
сохранить в json-формате.

    *Например, если в `parameters` содержаться инстансы регуляризаторов,
    в json можно положить соответствующие имена классов для каждого регуляризатора
    и словарь всех параметров.*

---

## Что нужно для создания нового кубика?

При выполнении всех этих условий у вас получится рабочий кубик, встраиваемый в библиотеку:

1. Новый кубик наследуется от BaseCube.

2. В классе-наследнике задаются методы `__init__`, `apply`, `get_jsonable_from_parameters`.
Метод `__call__` переопределять не рекомендуется.

3. `get_jsonable_from_parameters()[i]` соответствует тому же шагу кубика, что и `parameters[i]`.