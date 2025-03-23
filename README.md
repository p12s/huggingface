# huggingface
Это перевод, оригинал ![тут](https://github.com/domschl/HuggingFaceGuidedTourForMac?tab=readme-ov-file)

# Deep Learning гайд от HuggingFace для Mac с Apple Silicon

Обзор того, как установить оптимизированный `pytorch` и опционально новый `MLX` от Apple и/или `tensorflow` или `JAX` от Google на Apple Silicon Mac и как использовать большие языковые модели `HuggingFace` для собственных экспериментов. Последние Mac показывают хорошую производительность для задач машинного обучения.

Выполним следующие шаги:

- Установка `homebrew` 
- Установка `pytorch` с поддержкой MPS (metal performance shaders) с использованием графических процессоров Apple Silicon
- Установка нового фреймворка `mlx` от Apple ![Необязательно](http://img.shields.io/badge/optional-brightgreen.svg?style=flat)
- Установка `JAX` с драйверами Metal от Apple (экспериментальные на данный момент (2025-04), и не всегда актуальные.) ![Необязательно](http://img.shields.io/badge/optional-brightgreen.svg?style=flat)
- Установка `tensorflow` с оптимизацией подключаемого драйвера Metal от Apple ![Необязательно](http://img.shields.io/badge/legacy-optional-brightgreen.svg?style=flat)
- Установка `jupyter lab` для запуска Jupyter notebook
- Установка `huggingface` и запустите несколько предварительно обученных языковых моделей с помощью `transformers` и всего нескольких строк кода в jupyter lab.

Дополнительно:

- Запуск больших языковых моделей (LLM), которые конкурируют с коммерческими проектами: Llama 2 или Llama 3 с llama.cpp (s.b.) с использованием ускорения Mac Metal.

## Дополнительные заметки ![Необязательно](http://img.shields.io/badge/optional-brightgreen.svg?style=flat)

(перейдите к **1. Подготовка**, если вы знаете, какой фреймворк вы собираетесь использовать)  

### Что такое Tensorflow, JAX, Pytorch и MLX и как Huggingface со всем этим связан?  

Tensorflow, JAX, Pytorch и MLX — это фреймворки глубокого обучения, которые предоставляют необходимые библиотеки для выполнения оптимизированных тензорных операций, используемых при обучении и выводе. На высоком уровне функциональность всех четырех эквивалентна. Huggingface строится на основе любого из этих фреймворков и предоставляет большую библиотеку предварительно обученных моделей для множества различных вариантов использования, готовых к использованию или настройке, а также ряд удобных библиотек и примеров кода для легкого начала работы.  

- **Pytorch** — это наиболее общий и в настоящее время наиболее широко используемый фреймворк глубокого обучения. В случае сомнений используйте Pytorch. Он поддерживает множество различных аппаратных платформ (включая оптимизации Apple Silicon).  
- **JAX** — это более новый фреймворк Google, который исследователи считают лучшей альтернативой Tensorflow. Он поддерживает GPU, TPU и фреймворк Apple Metal (все еще экспериментальный) и является более «низкоуровневым», особенно при использовании без дополнительных слоев нейронной сети, таких как [flax](https://github.com/google/flax). JAX на Apple Silicon все еще «экзотичен», поэтому для производственных проектов используйте Pytorch, а для исследовательских проектов интересны как JAX, так и MLX: MLX имеет более динамичную разработку (на данный момент), JAX поддерживает больше аппаратных фреймворков (GPU и TPU) помимо Apple Silicon, но разработка драйверов `jax-metal` не всегда соответствует последним версиям `JAX`.  
- **MLX** — это новичок Apple, и поэтому общая поддержка и документация (в настоящее время) гораздо более ограничены, чем у других основных фреймворков. Он красивый и хорошо спроектирован (они извлекли уроки из Torch и TensorFlow), но при этом тесно связан с Apple Silicon. В настоящее время он лучше всего подходит для студентов, у которых есть оборудование Apple и которые хотят изучать или экспериментировать с глубоким обучением. То, чему вы учитесь с MLX, легко переносится в Pytorch, но имейте в виду, что для развертывания всего, что вы разработали, в не-Apple-вселенной необходимо преобразование моделей и перенос кода обучения и вывода.  
- **corenet** — это [недавно выпущенная библиотека обучения] Apple (https://github.com/apple/corenet), которая использует PyTorch и инфраструктуру HuggingFace, а также содержит примеры того, как переносить модели в MLX. См. пример: [OpenElm (MLX)](https://github.com/apple/corenet/blob/main/mlx_examples/open_elm).  
- **Tensorflow** — это «COBOL» глубокого обучения, и он практически молчаливо прекращен Google. Сам Google публикует новые модели для PyTorch и JAX/Flax, а не для Tensorflow. Если вас не заставляют использовать Tensorflow, потому что ваша организация уже использует его, игнорируйте его. Если ваша организация использует TF, составьте план миграции! Посмотрите на Pytorch для производства и JAX для исследований.  

HuggingFace публикует [Обзор поддержки моделей](https://huggingface.co/docs/transformers/index#supported-frameworks) для каждого фреймворка. В настоящее время Pytorch является стандартом де-факто, если вы хотите использовать существующие модели.  

> ![Примечание:](http://img.shields.io/badge/📝-Note:-green.svg?style=flat) Для (вероятно, слишком упрощенного) ответа на вопрос «Что самое быстрое?» взгляните на блокнот Jupyter [02-Бенчмарки](https://github.com/p12s/huggingface/blob/main/02-Benchmarks.ipynb), и после завершения установки вы можете протестировать свою собственную среду. Блокнот позволяет сравнивать скорость умножения матриц для разных фреймворков. Однако разница между фреймворками при выполнении «стандартных» задач обучения или вывода моделей, скорее всего, будет менее выраженной.  

## 1. Подготовка

### 1.1 Установка homebrew

Если вы этого еще не сделали, перейдите на <https://brew.sh/> и следуйте инструкциям по установке homebrew.  
После этого откройте терминал и введите `brew --version`, чтобы проверить, что он установлен правильно.  
Теперь используйте `brew` для установки более новых версий `python` и `git`. Рекомендуется использовать Python 3.12 по умолчанию для Homebrew, если вы не планируете использовать Tensorflow с оптимизацией Metal (все еще требуется 3.11 (в 2024-04)).  

#### Текущий Python для Huggingface, Pytorch, JAX и MLX, Python 3.12, Homebrew по умолчанию

```bash
brew install python@3.12 git
```

#### Устаревшие установки (Tensorflow), Python 3.11 ![Необязательно](http://img.shields.io/badge/optional-brightgreen.svg?style=flat)

```bash
brew install python@3.11 git
```

> ![Примечание:](http://img.shields.io/badge/📝-Note:-green.svg?style=flat) вы можете установить обе версии Python, а затем создать виртуальную среду, используя нужную вам версию Python для каждого случая.

> ![Примечание:](http://img.shields.io/badge/📝-Note:-green.svg?style=flat) Если вы также планируете использовать Linux, учтите, что поддержка версий Python иногда различается между версиями фреймворков для Mac и Linux.

#### Сделайте Python от Homebrew системным по умолчанию![Необязательно](http://img.shields.io/badge/optional-brightgreen.svg?style=flat)

> ![Примечание:](http://img.shields.io/badge/📝-Note:-green.svg?style=flat) Apple не вкладывает много сил в поддержание актуальности Python от MacOS. Если вы хотите использовать актуальный Python по умолчанию, имеет смысл сделать Python от Homebrew системным Python по умолчанию.
Итак, если вы хотите использовать homebrew's Python 3.11 или 3.12 системно-глобально, самый простой способ сделать это (после `brew install python@3.12` или `3.11`):

Отредактируйте `~/.zshrc` и вставьте:

```bash
# Это НЕОБЯЗАТЕЛЬНО и требуется только если вы хотите сделать homebrew's Python 3.12 глобальной версией:
export PATH="/opt/homebrew/opt/python@3.12/bin:$PATH"
export PATH=/opt/homebrew/opt/python@3.12/libexec/bin:$PATH
```

Измените все ссылки `3.12` на `3.11`, если хотите сделать homebrew's Python 3.11 системно-стандартным python.

(Перезапустите терминал, чтобы активировать изменения пути, или введите `source ~/.zshrc` в текущем сеансе терминала.)

> ![Примечание:](http://img.shields.io/badge/📝-Note:-green.svg?style=flat) Независимо от используемого системного python при создании виртуальной среды вы всегда можете выбрать конкретную версию python, которую хотите использовать в `venv`, создав `venv` именно с этим python. Например, `/usr/bin/python3 -m venv my_venv_name` создает виртуальную среду с использованием python macOS от Apple (который на момент написания этой статьи, 2025-04, все еще застрял на версии 3.9.6). Подробнее см. ниже, **Виртуальные среды**.

### 1.2 Тестовый проект

Теперь клонируйте этот проект как тестовый проект:

```bash
git clone https://github.com/p12s/huggingface
```

Это клонирует тестовый проект в каталог `huggingface`

#### Виртуальная среда

Теперь создайте среду Python 3.12 для этого проекта и активируйте ее:

(Снова: замените на `3.11`, если нужно)

```bash
python3.12 -m venv huggingface
```

Создание venv добавляет необходимые файлы (бинарные файлы Python, библиотеки, конфигурации) для виртуальной среды Python в папку проекта, которую мы только что клонировали, снова используя тот же каталог `huggingface`. Войдите в каталог и активируйте виртуальную среду:

```bash
cd huggingface
source bin/activate
```

Теперь каталог `huggingface` содержит содержимое репозитория github (например, `00-SystemCheck.ipynb`) _и_ файлы для виртуальной среды (например, `bin`, `lib`, `etc`, `include`, `share`, `pyvenv.cfg`):

![Содержимое папки](https://github.com/p12s/huggingface/blob/main/Resources/ProjectFolder.png)

**Альтернативы:** Если у вас установлено много разных версий Python, вы можете создать среду, которая использует определенную версию, указав путь к Python, который используется для создания `venv`, например:

```bash
/opt/homebrew/opt/python@3.12/bin/python3.12 -m venv my_new_312_env
```

явно использует Python Homebrew для создания нового `venv`, тогда как

```bash
/usr/bin/python3 -m venv my_old_system_venv
```

использует версию Python MacOS от Apple для новой среды.

### 1.3 Когда вы закончите свой проект

Деактивируйте эту виртуальную среду, просто используйте:

```bash
deactivate
```

Чтобы повторно активировать ее, войдите в каталог, содержащий `venv`, здесь: `huggingface` и используйте:

```bash
source bin/activate
```

### Дополнительные примечания по `venv` ![Необязательно](http://img.shields.io/badge/optional-brightgreen.svg?style=flat)
> ![Предупреждение](http://img.shields.io/badge/⚠️-Warning:-orange.svg?style=flat) Очень **неинтуитивное свойство `venv`** заключается в следующем: когда вы входите в среду, активируя ее в подкаталоге вашего проекта (с помощью `source bin/activate`), `venv` **остается активным**, когда вы покидаете папку проекта и начинаете работать над чем-то совершенно другим _пока вы явно не деактивируете_ `venv` с помощью `deactivate`.
>
> Существует ряд инструментов, которые изменяют системное приглашение терминала для отображения текущего активного `venv`, что очень полезно. Посмотрите [starship](https://github.com/starship/starship) (рекомендуется) или, если вам нравится украшательство [`Oh My Zsh`](https://ohmyz.sh/).

![Нет активного venv](https://github.com/p12s/huggingface/blob/main/Resources/no_venv.png)
_Пример с установленным `powerlevel10k`. В левой части системного приглашения отображается текущий каталог, в правой части — имя `venv`. В настоящее время `venv` не активен._

После активации `venv` в `huggingface`:

![venv все еще активен](https://github.com/p12s/huggingface/blob/main/Resources/venv_remind.png)
_Даже если рабочий каталог изменен (здесь на `home`), так как `venv` все еще активен, его имя отображается справа с помощью `powerlevel10k`. Очень удобно._

> ![Примечание:](http://img.shields.io/badge/📝-Note:-green.svg?style=flat) Подробнее о виртуальных средах Python см. <https://docs.python.org/3/tutorial/venv.html>.

### 2 Установка `pytorch`

Убедитесь, что ваша виртуальная среда активна с помощью `pip -V` (заглавная буква V), это должно показать путь для `pip` в вашем проекте:

`<your-path>/huggingface/lib/python3.12/site-packages/pip (python 3.12)`

После `https://pytorch.org` мы установим Pytorch с помощью `pip`. Вам нужна как минимум версия 2.x (по умолчанию с 2023 года), чтобы получить поддержку MPS (Metal Performance Shaders) в pytorch, которая обеспечивает значительное преимущество в производительности на Apple Silicon.

Чтобы установить `pytorch` в `venv`:

```bash
pip install -U torch numpy torchvision torchaudio
```

#### 2.1 Быстрая проверка pytorch

Чтобы проверить, что `pytorch` установлен правильно и доступны шейдеры производительности MPS metal, откройте терминал, введите `python` и в оболочке python введите:

```python
import torch
# проверьте, доступен ли MPS:
torch.backends.mps.is_available()
```

Это должно вернуть `True`.

### 3 Установите `MLX` ![Необязательно](http://img.shields.io/badge/optional-brightgreen.svg?style=flat)

```bash
pip install -U mlx
```

#### 3.1 Быстрая проверка MLX

Снова запустите `python` и введите:

```python
import mlx.core as mx
print(mx.__version__)
```

Это должно вывести версию, например `0.16.1` (2024-07)

- Посетите Apple [проект MLX](https://github.com/ml-explore/) и особенно [mlx-examples](https://github.com/ml-explore/mlx-examples)!
- На Huggingface есть активное сообщество MLX, которое перенесло множество сетей на MLX: [Huggingface MLX-Community](https://huggingface.co/mlx-community)
- Новый [corenet](https://github.com/apple/corenet) от Apple использует PyTorch и инфраструктуру HuggingFace, а также содержит примеры того, как переносить модели на MLX. Смотрите пример: [OpenElm (MLX)](https://github.com/apple/corenet/blob/main/mlx_examples/open_elm).

## 4.1 Установите `JAX` ![Необязательно](http://img.shields.io/badge/optional-brightgreen.svg?style=flat)

JAX — отличный выбор, если вы сосредоточены на низкоуровневой оптимизации алгоритмов и исследованиях за пределами устоявшихся алгоритмов глубокого обучения. Созданный по образцу `numpy`, он поддерживает [автоматическую дифференциацию](https://jax.readthedocs.io/en/latest/jax-101/04-advanced-autodiff.html) «всего» (для задач оптимизации) и поддерживает [векторизацию](https://jax.readthedocs.io/en/latest/jax-101/03-vectorization.html) и [параллелизацию](https://jax.readthedocs.io/en/latest/jax-101/06-parallelism.html) алгоритмов Python за пределами простого глубокого обучения. Чтобы получить функциональность, ожидаемую от других фреймворков глубокого обучения (слои, функции цикла обучения и подобные «высокоуровневые»), рассмотрите возможность установки дополнительной библиотеки нейронных сетей, такой как: [`flax`](https://github.com/google/flax).

### Проверьте поддерживаемые версии

К сожалению, драйверы `JAX` metal начали отставать от релизов JAX, поэтому вам необходимо проверить [таблицу совместимости](https://pypi.org/project/jax-metal/) на предмет поддерживаемых версий `JAX`, которые соответствуют доступным драйверам `jax-metal`.

Чтобы установить определенную версию `JAX` и последнюю `jax-metal` с `pip` в активную среду:

```bash
# Версия 0.4.26 взята из таблицы совместимости, упомянутой выше. Обновляйте по мере необходимости.
pip install -U jax==0.4.34 jaxlib==0.4.34 jax-metal
```

#### 4.2 Быстрая проверка JAX

Запустите `python` (поддерживается 3.12) и введите:

```python
import jax
print(jax.devices()[0])
```

Должно отобразиться (только при первом запуске):

```
Платформа 'METAL' является экспериментальной, и не все функции JAX могут поддерживаться правильно!

ВНИМАНИЕ: все сообщения журнала до вызова absl::InitializeLog() записываются в STDERR
W0000 00:00:1721975334.430133 43061 mps_client.cc:510] ВНИМАНИЕ: Поддержка JAX Apple GPU является экспериментальной, и не все функции JAX могут поддерживаться правильно!
Устройство Metal установлено на: Apple M2 Max

systemMemory: 32,00 ГБ
maxCacheSize: 10,67 ГБ

I0000 00:00:1721975334.446739 43061 service.cc:145] Служба XLA 0x60000031d100 инициализирована для платформы METAL (это не гарантирует, что будет использоваться XLA). Устройства:
I0000 00:00:1721975334.446771 43061 service.cc:153] Устройство StreamExecutor (0): Metal, <undefined>
I0000 00:00:1721975334.448269 43061 mps_client.cc:406] Используется простой распределитель.
I0000 00:00:1721975334.448308 43061 mps_client.cc:384] XLA бэкэнд будет использовать до 22906109952 байт на устройстве 0 для SimpleAllocator.
[METAL(id=0)]
```

Здесь `METAL:0` — это устройство, которое JAX будет использовать для вычислений, и Apple Silicon поддерживается.

##### Ошибки

Если вместо этого вы видите ошибки типа:

```
RuntimeError: Невозможно инициализировать бэкенд 'METAL': INVALID_ARGUMENT: Несоответствие версии API PJRT плагина PJRT (0.47) и версии API PJRT фреймворка 0.54). (Вам может потребоваться удалить неисправный пакет плагина или задать JAX_PLATFORMS=cpu, чтобы пропустить этот бэкенд.)
```

Ваши версии `jax` и `jaxlib` несовместимы с `jax-metal`. Проверьте [таблицу совместимости](https://pypi.org/project/jax-metal/) на предмет `jax-metal` и установите требуемые версии, как указано в таблице.

- [Примеры проектов HuggingFace с JAX и Flax](https://github.com/huggingface/transformers/tree/main/examples/flax)
- Довольно лаконичная документация Apple находится в [документации Apple JAX](https://developer.apple.com/metal/jax/).

## 4.3 Установите `tensorflow` ![Необязательно](http://img.shields.io/badge/legacy-optional-brightgreen.svg?style=flat)

> ![Предупреждение:](http://img.shields.io/badge/⚠-Note:-orange.svg?style=flat) Tensorflow быстро теряет поддержку, и даже Google не публикует новые модели для Tensorflow. Рекомендуется план миграции, если вы планируете использовать это.

> ![Примечание:](http://img.shields.io/badge/📝-Note:-green.svg?style=flat) Хотя Tensorflow поддерживает Python 3.12 с версии 2.16, ускоритель macOS `tensorflow-metal` не обновлялся с 2023-09 (состояние 2025-04) и требует Python 3.11:

Убедитесь, что ваша виртуальная среда активна с `pip -V` (заглавная V), это должно показать путь для `pip` в вашем проекте:

`<your-path>/huggingface/lib/python3.11/site-packages/pip (python 3.11)`

Следуя <https://developer.apple.com/metal/tensorflow-plugin/>, мы установим `tensorflow` с помощью `pip` в нашем `venv`:

```bash
pip install -U tensorflow tensorflow-metal
```

#### 4.4 Быстрая проверка Tensorflow

Чтобы проверить, что `tensorflow` установлен правильно, откройте терминал, введите `python` и в оболочке python введите:

```python
import tensorflow as tf
tf.config.list_physical_devices('GPU')
```

Вы должны увидеть что-то вроде:
```
[PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```

### 5 Лаборатория Jupyter

На этом этапе ваш Apple Silicon Mac должен быть готов к запуску `pytorch` и, по желанию, `MLX` и/или `JAX` или `tensorflow` с поддержкой аппаратного ускорения, используя фреймворк Apple Metal.

Чтобы проверить это, вы можете использовать `jupyter lab` для запуска некоторых блокнотов. Чтобы установить `jupyter lab`, сначала убедитесь, что виртуальная среда, которую вы хотите использовать, активна (`pip -V`), и введите:

```bash
pip install -U jupyterlab ipywidgets
```

> ![Примечание:](http://img.shields.io/badge/📝-Note:-green.svg?style=flat) Если у вас установлены другие версии Jupyter, путь к недавно установленной версии Jupyter в `venv` часто обновляется неправильно, повторно активируйте среду, чтобы убедиться, что используется правильная локальная версия Jupyter:

```bash
deactivate
source bin/activate
```

Чтобы запустить Jupyter lab, введите:

```bash
jupyter lab
```

Должно открыться окно браузера с запущенной `jupyter lab`. Затем вы можете создать новый блокнот Python и запустить код, чтобы проверить, что `tensorflow` и `pytorch` работают правильно:

![](Resources/jupyterlab.png)

```python
import torch

print("Pytorch version:", torch.__version__)
```

Если все прошло успешно, ваш Mac готов к экспериментам с глубоким обучением.

## 6 HuggingFace

HuggingFace — отличный ресурс для экспериментов с NLP и глубоким обучением. Он предоставляет большое количество предварительно обученных языковых моделей и простой API для их использования. Он позволит нам быстро приступить к экспериментам с глубоким обучением.

### 6.1 Установка `transformers`

В [инструкциях по установке huggingface](https://huggingface.co/docs/transformers/installation) мы используем `pip` для установки `transformers`:

```bash
pip install -U transformers acceleration "huggingface_hub[cli]"
```

> ![Примечание:](http://img.shields.io/badge/📝-Note:-green.svg?style=flat) При экспериментах с HuggingFace вы загрузите большие модели, которые будут храниться в вашем домашнем каталоге по адресу: `~/.cache/huggingface/hub`.
> Вы можете удалить эти модели в любое время, удалив этот каталог или часть его содержимого.

- `accelerate` необязателен, но используется для запуска некоторых больших моделей. Побочным эффектом установки `accelerate` может стать понижение версии некоторых других модулей, например `numpy`.

- `"huggingface_hub[cli]"` устанавливает инструменты командной строки huggingface, которые иногда требуются для загрузки (частично лицензированных) моделей, например Llama 3.

## 7 Эксперименты

### 7.1 Простой анализ настроений

В каталоге `huggingface` и активном `venv` запустите `jupyter lab` и загрузите блокнот `00-SystemCheck.ipynb`. Сначала блокнот проверит все фреймворки глубокого обучения и предоставит информацию, если они правильно установлены. После этого для простого эксперимента используется Pytorch.

Используйте `<Shift>-Enter` для запуска ячеек блокнота.

> ![Примечание:](http://img.shields.io/badge/📝-Note:-green.svg?style=flat) Если вы запустили Jupyter Lab до установки Huggingface, вам нужно либо перезапустить ядро ​​python в Jupyter, либо просто перезапустить Jupyter Lab, иначе он не найдет библиотеку Transformers.

После различных тестов вы должны наконец увидеть что-то вроде этого:

![](Resources/huggingface-transformers.png)

Если вы получили классификацию метки `ПОЛОЖИТЕЛЬНЫЙ` с оценкой `0,99`, то вы готовы начать экспериментировать с HuggingFace!

> ![Примечание:](http://img.shields.io/badge/📝-Note:-green.svg?style=flat) Вы увидите, что библиотеки `HuggingFace` загружают всевозможные большие двоичные блоки, содержащие данные обученной модели. Эти данные хранятся в вашем домашнем каталоге по адресу: `~/.cache/huggingface/hub`. Вы можете удалить эти модели в любое время, удалив этот каталог или части его содержимого.

#### Устранение неполадок

- Если самотестирование не удалось ('xyz не найден!'), убедитесь, что pytorch, jax (необязательно), MLX (необязательно), tensorflow (необязательно), jupyter и transformers by huggingface установлены в одной и той же активной виртуальной среде Python, иначе компоненты не будут 'видеть' друг друга!

### 7.2 Минимальный чат-бот

Вы можете открыть блокнот [`01-ChatBot.ipynb`](https://github.com/p12s/huggingface/blob/main/01-ChatBot.ipynb), чтобы попробовать очень простого чат-бота на вашем Mac.

Используемый код Python:

```python
import torch 
from transformers import AutoModelForCausalLM, AutoTokenizer
from transformers.utils import logging

# Disable warnings about padding_side that cannot be rectified with current software:
logging.set_verbosity_error()

model_names = ["microsoft/DialoGPT-small", "microsoft/DialoGPT-medium", "microsoft/DialoGPT-large"]
use_model_index = 1  # Change 0: small model, 1: medium, 2: large model (requires most resources!)
model_name = model_names[use_model_index]
          
tokenizer = AutoTokenizer.from_pretrained(model_name) # , padding_side='left')
model = AutoModelForCausalLM.from_pretrained(model_name)

# The chat function: received a user input and chat-history and returns the model's reply and chat-history:
def reply(input_text, history=None):
    # encode the new user input, add the eos_token and return a tensor in Pytorch
    new_user_input_ids = tokenizer.encode(input_text + tokenizer.eos_token, return_tensors='pt')

    # append the new user input tokens to the chat history
    bot_input_ids = torch.cat([history, new_user_input_ids], dim=-1) if history is not None else new_user_input_ids

    # generated a response while limiting the total chat history to 1000 tokens, 
    chat_history_ids = model.generate(bot_input_ids, max_length=1000, pad_token_id=tokenizer.eos_token_id)

    # pretty print last ouput tokens from bot
    return tokenizer.decode(chat_history_ids[:, bot_input_ids.shape[-1]:][0], skip_special_tokens=True), chat_history_ids

history = None
while True:
    input_text = input("> ")
    if input_text in ["", "bye", "quit", "exit"]:
        break
    reply_text, history_new = reply(input_text, history)
    history=history_new
    if history.shape[1]>80:
        old_shape = history.shape
        history = history[:,-80:]
        print(f"History cut from {old_shape} to {history.shape}")
    # history_text = tokenizer.decode(history[0])
    # print(f"Current history: {history_text}")
    print(f"D_GPT: {reply_text}")
```

This shows a (quite limited and repetitive) chatbot using Microsoft's [DialoGPT](https://huggingface.co/microsoft/DialoGPT-medium?text=Hey+my+name+is+Mariama%21+How+are+you%3F) models.

Things to try:

- By changing `use_model_index` between `0..2`, you can select either a small, medium or large language model.
- To see the history that the model maintains you can uncomment the two `history_text` related lines above.
- To get rid of the downloaded models, clean up `~/.cache/huggingface/hub`. Missing stuff is automatically re-downloaded when needed.

## Следующие шаги

- Ваш Mac может запускать большие языковые модели, которые по производительности соперничают с коммерческими решениями. Прекрасным примером является проект [`llama.cpp`](https://github.com/ggerganov/llama.cpp), который реализует код вывода, необходимый для запуска LLM в высокооптимизированном коде C++, поддерживающем ускорение Metal для Mac.<br>Пошаговое руководство по компиляции и запуску Llama 3 или Llama 2 сначала для бенчмаркинга, а затем для чата можно найти здесь:<br>[Llama.cpp чат с использованием модели Llama 2, с первой поддержкой Llama 3](https://github.com/p12s/huggingface/blob/main/NextSteps/llama.cpp.md). Кроме того, предоставляется первая версия для Llama 3.

## Ресурсы для обучения

- Один из лучших (на данный момент) источников информации о новых выпусках моделей на Huggingface — [группа Reddit LocalLLama](https://old.reddit.com/r/LocalLLaMA/).
- Быстрый способ узнать, как на самом деле работают модели нейронных сетей и, в частности, больших языков, — это курс Андрея Карпати на Youtube: [Подробное введение в нейронные сети и обратное распространение: создание микрограда](https://www.youtube.com/watch?v=VMj-3S1tku0&list=PLAqhIrjkxbuWI23v9cThsA9GvCAUhRvKZ). Если вы немного знаете Python и знаете, как умножать матрицы с помощью NumPy, этот курс поможет вам полностью научиться создавать собственную модель большого языка с нуля.
