# StaticJinjaPlus

StaticJinjaPlus is a tool to build static sites using [Jinja](https://jinja.palletsprojects.com/).

# Content

- [How to install](#How-to-install)
- [Building sites](#Building-sites)
  - [Watching for changes](#Watching-for-changes)
  - [Specifying templates or build paths](#Specifying-templates-or-build-paths)
  - [Using assets](#Using-assets)
  - [Using context](#Using-context)
  - [Шаблоны extends и include](#Шаблоны-extends-и-include)
- [Example templates](#Example-templates)
- [Использование Docker](#Docker-) 

## How to install

Python should already be installed. This project requires Python3.7 or newer.

Clone the repo / download code.

Using virtual environment [virtualenv/venv](https://docs.python.org/3/library/venv.html) is recommended for project isolation.

Install requirements:
```commandline
pip install -r requirements.txt
```

To check that everything installed correctly try running the script with `--help` flag:
```commandline
python main.py --help
```
Output:
```
usage: main.py [-h] [-w] [--srcpath SRCPATH] [--outpath OUTPATH]

Render HTML pages from Jinja2 templates

options:
  -h, --help         show this help message and exit
  -w, --watch        Render the site, and re-render on changes to <srcpath>
  --srcpath SRCPATH  The directory to look in for templates (defaults to './templates)'
  --outpath OUTPATH  The directory to place rendered files in (defaults to './build')
```
Now you're all ready to build your static sites!


## Building sites

Note: see [Example templates](#example-templates) section for an example with sample templates. Rename the /templates_example folder to /templates to run test templates.

To render html pages from templates, run:
```commandline
python main.py
```
This will look in `./templates` for templates (files whose name does not start with `.` or `_`) and build them to `./build`.

You'll see a message about each template in the output:
```
Rendering index.html...
```

### Watching for changes
To watch for changes in the templates and recompile build files if changes happen, use `-w` or `--watch` argument.
```commandline
python main.py -w

Rendering index.html...
Watching '/home/mrdave/Python Projects/StaticJinjaPlus/templates' for changes...
Press Ctrl+C to stop.
```

### Specifying templates or build paths

To change the source and/or output paths use optional arguments:  
- `--srcpath` - the directory to look in for templates (defaults to `./templates`)  
- `--outpath` - the directory to place rendered files in (defaults to `.`)

Example:
```commandline
python main.py --srcpath other_template_folder --outpath ./built/site

Rendering index.html...
```

### Using assets

To use assets such as `.css` and `.js` files with your templates, place them in `assets` folder inside source path (so `./templates/assets` by default).
In this case StaticJinjaPlus will copy the assets to output folder keeping the same relative paths.

Building log will also include "rendering" the assets:

```commandline
python main.py

Rendering assets/style.css...
Rendering index.html...
```

### Using context
It is possible to pass context for use in your templates by setting environmental variables named as you use them in the templates with the `"SJP_"` prefix.

As an example, if your template includes `thing` variable, pass the `SJP_THING` env variable before building.

```html
<!-- html template -->
<div class="container">
    <h1>Welcome to our website!</h1>
    <p>This is the homepage content. Replace it with your own.</p>
    <p>The thing from context is {{ thing }}</p>
</div>
```
```shell
export SJP_THING="my_thing"
python main.py
```
![](https://imgur.com/TEf3yJ6.png)


## Шаблоны extends и include

- Вызов `extends` используется когда шаблоны должны иметь одинаковую базовую структуру, одну и ту же разбивку по блокам, но с разным содержимым для этих блоков. Это позволяет сформировать единообразный стиль сайта, когда веб-страницы имеют одни и те же структурные элементы - меню, хедер, футер, сайдбары и так далее.

Пример кода:

```html
<!-- index.html file -->

{% extends '_base.html' %}
```

- Вызов `include` добавляет в нужное место кусок шаблона. Название подключаемого шаблона передается в качестве параметра.
  
Пример вызова `include` для файла `_card.html` из `index.html`. Имя файла `_card.html` имеет префикс «`_`» что объявляет его вспомогательным (подключаемым). При изменении вспомогательного файла рендерятся все файлы в которых есть к нему обращения. В данной конструкции передадим еще две переменные `page` и  `number`.

```html
<!-- index.html file -->

{% with page="Домашняя", number=1 %}
  {% include "_card.html" %}
{% endwith %}
```

Пример кода с переменными `page` и `number`.
  
```html
<!-- _card.html file -->

<p>Вывод текста из файла _card.html методом include. Страница {{page}} Номер {{number}} </p>
```

<img width="738" alt="image" src="https://github.com/SGKespace/StaticJinjaPlus/assets/55636018/6fbce118-e5ae-46b8-b5e5-7ab4df323562">


## Example templates
The repository has example templates to see how StaticJinjaPlus works.

Run the following command and see your results in `./build`:
```commandline
python main.py --srcpath example/templates
```
```shell
build
├── about.html
├── assets
│  └── style.css
├── faq.html
└── index.html

```
![Example of index.html](https://imgur.com/Onr3aVM.jpg)
Example render of `index.html`

## Работа с Docker

### Шаги для запуска контейнера и сборки статического сайта:
[Ссылка на образы](https://hub.docker.com/repository/docker/barseeek/static-jinja-plus/general)
1. Убедитесь, что Docker установлен:
   - Если у вас еще не установлен Docker, загрузите и установите его с официального сайта Docker (https://www.docker.com/).

2. Войдите в Docker Hub:
    ```bash
    docker login --username=your_dockerhub_username --password=your_dockerhub_password
    ```

3. Загрузите образ из Docker Hub (если это еще не сделано):
    ```bash
   docker pull barseeek/static-jinja-plus:tagname
   ```  
    
4. Запустите контейнер:
    ```bash
   cd /path/to/project_folder/
   docker run --rm -v "$(pwd)/templates:/WORKDIR/templates" -v "$(pwd)/output:/WORKDIR/build" barseeek/static-jinja-plus:tagname
   ```
   - `WORKDIR` для образов ubuntu `/opt/StaticJinjaPlus/`, для slim `/StaticJinjaPlus/`
   - `--rm`: удалить контейнер после завершения работы.
   - `-v "$(pwd)/templates:/StaticJinjaPlus/templates"`: смонтируйте папку templates с вашего локального компьютера в контейнер, чтобы docker мог получить доступ к вашим шаблонам и ассетам.
   - `-v "$(pwd)/build:/StaticJinjaPlus/build"`: смонтируйте папку `build` с вашего локального компьютера в контейнер, чтобы вы могли получить результаты работы статического генератора сайтов.
   - `tagname` замените на один из [тэгов](https://hub.docker.com/repository/docker/barseeek/static-jinja-plus/tags) из Docker Hub.
5. Проверка выходных данных:
   
После завершения работы контейнера сгенерированные статические файлы будут доступны в папке `templates/build` (если StaticJinjaPlus использует стандартные настройки).

### Аргументы для создания образа
В Dockerfilах проекта аргументы объявляются следующим образом:
```bash
ARG SJP_COMMIT=main
ARG TEMPLATE_FOLDER=templates
```
SJP_COMMIT задает конкретный коммит для клонирования репозитория, а TEMPLATE_FOLDER задает папку с шаблонами, которая будет скопирована в образ.
#### Пример создания образа
Образ `0.1.0-slim`:
```bash
docker build -t barseeek/static-jinja-plus:0.1.0-slim -f Dockerfiles/Dockerfile_slim . --build-arg SJP_COMMIT=a9f8f5ba28827841eab003c8b7d49d757f2df9e2 --build-arg TEMPLATE_FOLDER=new_templates
docker run -v "$(pwd)/build:/StaticJinjaPlus/build" -v "$(pwd)/new_templates:/StaticJinjaPlus/new_templates" -it barseeek/static-jinja-plus:0.1.0-slim
```