# Seo Pro: INSTALLATION

Tested on Opencart v1.5.6.4 (oc1564).
Should work on all oc15x versions.

This repository contains all modified files, so you can compare them with yours.
You can use Total Commander (Windows), WinMerge (Windows), Meld (Linux) to compare
files and folders.


* execute SQL query

        ALTER TABLE `product_to_category` ADD `main_category` tinyint(1) NOT NULL DEFAULT '0';

    if you use prefix, add it to the table name.

* copy file `seo_pro.php` into the folder `catalog/controller/common`

* compare and make changes to all other files.

    GUI tools that can help you compare files and directories:
    Total Commander (Windows), WinMerge (Windows), Meld (Linux).

* Open file `index.php`, find and replace this line

        $controller->addPreAction(new Action('common/seo_url'));

    with following

        if (!$seo_type = $config->get('config_seo_url_type')) {
            $seo_type = 'seo_url';
        }
        $controller->addPreAction(new Action('common/' . $seo_type));

# Дубли страниц, SeoPro

<http://opencartforum.ru/topic/10270-dubli-stranitc-seopro/>

Для того, чтобы избавиться от дублей на страницы товаров,
[Yesvik](http://opencartforum.ru/user/6876-yesvik/) написал SeoPro (он
включён в поставку ocStore).


## cached\_seo\_pro

Этот мод -- дополнение к его замечательному решению. Позволяет две вещи:

1. ЧПУ на любой route
2. Кеширование запросов к таблице `url_alias`

что даёт:

* некешируемый SeoPro: 45 запросов выполняются каждый раз при загрузке страницы
* кешируемый: запрос один, выполняется раз в час(время жизни кеша по умолчанию)

## seopro_multilang

даёт возможность ВКЛЮЧИТЬ КОД ВЫБРАННОГО ЯЗЫКА В URI



# Чем SeoPro отличается от стандартного SeoUrl

См. <http://opencartforum.ru/topic/20526-seopro-vs-seourl/>

**Q:**

> Объясните, пожалуйста, популярно, чем SEOpro лучше SEOurl?
> На opencart 1.5.5.1 SEOurl входит в поставку. Включил, вроде работает... Стоит ли ставить pro?
> Какие дополнительные плюсы он даст?
>
> И по дублям - кто-то пишет, что SEOurl не создает дублей, в отличие
> от SEOpro.. С другой стороны, вроде бы SEOpro как раз и создавался
> для борьбы с дублями. Хотя откуда они могли появиться до установки ЧПУ...
> Или модули создают ЧПУ страницы и потом сами устраняют возникшие дубли?
> Опять же, с дублями, как я понимаю, вполне успешно можно бороться через
> robots.txt - зачем отдельный модуль с редиректами?
>
> Разложите по полочкам, пожалуйста!

**A:**

В опенкарт товар может принадлежать НЕСКОЛЬКИМ категориям.
В результате появляется неоднозначность и невозможность автоматически
формировать один-единственный (канонический) адрес страницы.

*   Как это решается в **SeoUrl**.

    Указанием CANONICAL в мета-тегах. У страниц в магазине могут встречаться
    на сайте разные адреса. Таким образом, поисковику сообщается, чьими
    дублями являются страницы, но физически эти разные URI на сайте
    присутствуют. В диагностике поисковых систем это будет указано. Многие
    воспринимают это как сообщение об ошибке и сотнях дублей.

    Каноническим считается адрес из URL + SEO-KEYWORD (без категорий,
    полного пути к товару и подобного). Во всевозможных ссылках на этот
    товар он указывается в мета-теге canonical.

*  Как это решается в **SeoPro**.

    Вводится понятие ОСНОВНОЙ КАТЕГОРИИ. Меняется механизм формирования
    ссылок. Таким образом, на сайте физически исключается наличие разных
    ссылок на один и тот же товар. Ссылки унифицированы и во всех местах
    сайта выглядят одинаково. За счет этого в диагностике поисковых систем
    для вебмастера пропадают эти сообщения о дублях (найденных где-то, но
    не включенных в индекс по причине canonical, указывающего на реальную
    страницу).

См. также:

* <http://opencartforum.ru/topic/2463-ustranenie-dublei-stranitc-tovarov-i-kategorii/>
* <http://opencartforum.ru/topic/10270-dubli-stranitc-seopro/>

## Если магазин установлен в папку, возможна ошибка при включенном SeoPro

Если возникает ошибка

* На этой странице обнаружена циклическая переадресация
* Ошибка 310 (net::ERR\_TOO\_MANY\_REDIRECTS): Обнаружено слишком много переадресаций.

См. <http://opencartforum.ru/topic/9542-pri-vkliuchenii-seopro-problemy/>

Проблема проявляется при установке в папку.
Если на хостинге магазин будет в корне сайта (домен, поддомен - неважно) проблем не будет.

**Yesvik**: Возможно проблема возникает при валидации... сейчас нет времени потестить при установке в папку.
Как вариант могу предложить следующее: в файле `catalog\controller\common\seo_pro.php`, после строки

	private function validate() {

добавить строку

	return;

это отключит валидацию... а при переносе на хостинг добавленную строку - удалить.

При этом не будет формироваться полная структура ссылки (`/category/subcategory/product`
при включенной соответсвующей опции) и работать переадресация (`store/product_name`
не будет редиректиться на требуемую ссылку `store/category/subcategory/product_name`,
хотя обе ссылки работают)

Причина: именно по итогам валидации принимается решение о редиректе.

**UPD (rb2):** по-моему, в этом репозитории эта ошибка уже исправлена.
