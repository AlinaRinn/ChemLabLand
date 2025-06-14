# Структура проекта #

## assets 
> ### fonts
Шрифты. Godot не поддерживает русский язык, поэтому каждый ярлык, текстбокс и тд. должен иметь прикрепленный шрифт с поддержкой языка. Частично реализовано через темы
> ### images
Изображения, используемые в проекте
> ### materials
Материалы. Можно создавать в самом движке или импортировать извне (рекомендуется)
> ### models
3D-модели 
> ### particle
Частицы
> ### textures 
Текстуры для создания материалов

## characters
Для хранения реализации игроков

## docs
Документация по проекту. Вы сейчас здесь =)

## objects
Префабы, т.е. обработанные и текстурированные модели, со скриптами, анимациями, коллайдерами, готовые к интеграции в уровень

## scenes 
> ### levels
Полноценные детализированные уровни, в которые происходит загрузка игрока
> ### main
Остальные вспомогательные сцены

## scripts
Общие скрипты, применяющиеся во многих местах проекта, а также некоторые скрипты объектов. Все остальные скрипты - вложенные. global.gd - синглтон, т.е. имеет видимость из всех сцен проекта.

<br><br>

# Godot #
Актуально для версий Godot 3.5.3 и ранее
## Импорт и обработка 3D-моделей 

### .fbx/.glb -> godot -> .tscn -> наследование в других сценах
-----
+ сохраняет иерархию и составную геометрию
- невозможно править иерархию свойства, вложенные объекты модели и тд
- тяжело добавлять новые дочерние/родительские объекты

### .gltf -> godot (built in в параметрах импорта) -> .tscn -> наследование в других сценах
-----
+ сохраняет иерархию, составную геометрию и !вложенные текстуры моделей
- не подходит для прозрачных моделей из-за собственного флага godot - transparent у материалов
- при клонировании репозитория проекта (без файлов .import, хранить которые не слишком целисообразно) годот потеряет все настройки; модели, импортированные таким образом, придется реимпортировать (за что)

### .fbx/.glb -> godot -> .tscn -> использование в других сценах
-----
+ сохраняет иерархию и составную геометрию
+ доступны любые изменения
- при создании сцены на основе модели, ее вес вырастает в 3-4 раза и сохраняется при создании экземпляров (при больших размерах моделей оптимизация покидает проект) 

### .obj -> godot -> .tscn 
-----
+ быстро, просто, мало весит, олрайт
+ можно непосредственно применять в сцене без префаба, т.к. сам формат .obj поддерживается нативно и редактируем
- не подходит для динамических объектов - теряется составная геометрия
- часто бывают проблемы и импортом/экспортом сложной геометрии (например, для некоторых типов отверстий и окружностей необходима триангуляция меша, искажающая модель)


## Особенности движка (как оживить инвалида) ##

+ До импорта модели в godot принципиально важно задать точку вращения (pivot/gizmo point), тк встроенного удобного инструмента для ее изменения в godot нет. Однако, точкой вращения годот считает начало координат, поэтому сложные составные объекты все равно нужно импортировать по частям. UPD. Можно задать точку вращения через наследование. Задаем нужные координаты оси вращения родителю (например, spital), тогда наследник вращается вокруг него.
+ UV развертки и материал лучше создавать в стороннем софте, так как встроенные возможности godot ограничены и не позволяют, например, вращать текстуру
+ Если сыпет ошибки текстур - не забываем прокликивать все составные сцены
+ Для корректного наследования без разрастания модели (как в случае .fbx/.glb -> godot -> .tscn -> использование в других сценах) в случаях с нетипичными корневыми нодами необходимо править настройку импорта
+ Ошибки импорта (Например: "_get_parent_class: Cannot get class 'FontFile'" или "Resource file not found: res://.godot/imported/myfont.ttf-xxxxxx.fontdata" или _load: Resource file not found: .) шрифтов правятся через удаления файлов .import (находятся в одном каталоге с исходными шрифтами) и перезапуск среды
+ Чтобы отключить "залупливание" аудио, нужно не только снять галочку, но и переимпортировать исходный файл
+ Менять свойства импортированных сцен (В древе серые, например, излучение материала импортированного светильник.gltf) НЕВОЗМОЖНО. Даже если Godot делает вид, что все ок
