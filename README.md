|Трасса "Озеро"|Трасса "Джунгли"|
|:--------:|:------------:|
|![Трасса "Озеро"](images/lake_track.png)|![Трасса "Джунгли"](images/jungle_track.png)|

## Описание проекта

В этом проекте используется нейронная сеть для клонирования поведения управления автомобилем. Это задача регрессии с учителем, где прогнозируются углы поворота руля на основе изображений дороги перед автомобилем.

Изображения были сняты с трёх разных камер (с центра, слева и справа автомобиля).

Модель основана на [модели NVIDIA](https://devblogs.nvidia.com/parallelforall/deep-learning-self-driving-cars/), которая доказала свою эффективность в этой области.

Поскольку проект включает обработку изображений, модель использует свёрточные слои для автоматической генерации признаков.

### Включенные файлы

- `model.py` Скрипт для создания и обучения модели.
- `drive.py` Скрипт для управления автомобилем. Можно использовать исходную версию или модифицировать её.
- `utils.py` Скрипт для вспомогательных функций (например, предварительная обработка и аугментация изображений).
- `model.h5` Веса модели.
- `environments.yml` Конфигурация среды Conda (для использования TensorFlow без GPU).
- `environments-gpu.yml` Конфигурация среды Conda (для использования TensorFlow с GPU).

Примечание: `drive.py` взят из [Udacity Behavioral Cloning project GitHub](https://github.com/udacity/CarND-Behavioral-Cloning-P3) с модификацией для управления дросселем.

## Быстрый старт

### Установка необходимых библиотек Python

Требуется [Anaconda](https://www.continuum.io/downloads) или [Miniconda](https://conda.io/miniconda.html) для настройки среды.

```python
# Использование TensorFlow без GPU
conda env create -f environment.yml

# Использование TensorFlow с GPU
conda env create -f environment-gpu.yml
```

Или можно вручную установить необходимые библиотеки (смотрите содержимое файлов `environment*.yml`) с помощью pip.

### Запуск предварительно обученной модели

Запустите [симулятор Udacity для автономного вождения](https://github.com/udacity/self-driving-car-sim), выберите сцену и нажмите кнопку "Autonomous Mode". Затем выполните следующий код:

```python
python drive.py model.h5
```

### Обучение модели

Необходима папка с данными, содержащая изображения для обучения.

```python
python model.py
```

Этот процесс создаст файл `model-<epoch>.h5`, если производительность на данной эпохе лучше предыдущей. Например, после первой эпохи будет создан файл `model-000.h5`.

## Архитектура модели

Дизайн сети основан на [модели NVIDIA](https://devblogs.nvidia.com/parallelforall/deep-learning-self-driving-cars/), которая использовалась для тестирования автономного управления. Она хорошо подходит для этой задачи.

Модель представляет собой глубокую свёрточную сеть, подходящую для задач классификации и регрессии изображений. Я внес несколько изменений, чтобы улучшить результаты и предотвратить переобучение:

- Нормализация входных изображений с помощью слоя Lambda для предотвращения насыщения и улучшения градиентов.
- Добавлен дополнительный слой Dropout после свёрточных слоёв для предотвращения переобучения.
- Используется функция активации ELU для всех слоёв, кроме выходного, чтобы добавить нелинейность.

Итоговая архитектура модели выглядит следующим образом:

- Нормализация изображений
- Свёрточный слой: 5x5, фильтры: 24, шаги: 2x2, активация: ELU
- Свёрточный слой: 5x5, фильтры: 36, шаги: 2x2, активация: ELU
- Свёрточный слой: 5x5, фильтры: 48, шаги: 2x2, активация: ELU
- Свёрточный слой: 3x3, фильтры: 64, шаги: 1x1, активация: ELU
- Свёрточный слой: 3x3, фильтры: 64, шаги: 1x1, активация: ELU
- Dropout (0.5)
- Полносвязный слой: нейронов: 100, активация: ELU
- Полносвязный слой: нейронов: 50, активация: ELU
- Полносвязный слой: нейронов: 10, активация: ELU
- Полносвязный слой: нейронов: 1 (выход)

## Предобработка данных

### Размер изображений

- Изображения обрезаются, чтобы исключить небо и переднюю часть автомобиля.
- Изображения изменяются до размеров 66x200 (3 канала YUV) в соответствии с моделью NVIDIA.
- Изображения нормализуются (данные делятся на 127.5 и вычитается 1.0).

## Обучение модели

### Аугментация изображений

Для обучения я использовал следующие техники аугментации:

- Случайный выбор центральных, левых или правых изображений.
- Для левых изображений угол поворота увеличивается на 0.2, для правых уменьшается на 0.2.
- Случайное отражение изображений по горизонтали.
- Случайное горизонтальное и вертикальное смещение с корректировкой угла поворота.
- Случайное добавление теней.
- Случайное изменение яркости изображения.

Примеры преобразованных изображений:

**Центральное изображение**

![Центральное изображение](images/center.png)

**Левое изображение**

![Левое изображение](images/left.png)

**Правое изображение**

![Правое изображение](images/right.png)

**Отражённое изображение**

![Отражённое изображение](images/flip.png)

**Смещённое изображение**

![Смещённое изображение](images/trans.png)

## Результаты

Модель может проходить трассу без столкновений с ограждениями.

## Ссылки
- Модель NVIDIA: https://devblogs.nvidia.com/parallelforall/deep-learning-self-driving-cars/
- Симулятор Udacity: https://github.com/udacity/self-driving-car-sim

