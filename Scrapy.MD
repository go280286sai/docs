### Scrapy
````
    pip install Scrapy
    scrapy startproject example
    cd example
    scrapy genspider example example.com
    scrapy crawl news_spider -o news.json
````
````
    import scrapy
    
    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            "https://quotes.toscrape.com/tag/humor/",
        ]
    
        def parse(self, response):
            for quote in response.css("div.quote"):
                yield {
                    "author": quote.xpath("span/small/text()").get(),
                    "text": quote.css("span.text::text").get(),
                }
    
            next_page = response.css('li.next a::attr("href")').get()
            if next_page is not None:
                yield response.follow(next_page, self.parse)
````
````
    scrapy shell "https://quotes.toscrape.com/page/1/"
    response.css("title").getall()
    ['<title>Quotes to Scrape</title>']
````
### Selenium + Scrapy
````
import scrapy
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
from scrapy.http import HtmlResponse

from olx.items import OlxItem


class UrlOlxSpider(scrapy.Spider):
    name = "url_olx"
    allowed_domains = ["olx.ua"]
    list_urls = [f"https://www.olx.ua/nedvizhimost/kvartiry/?currency=USD&page={i}" for i in range(2,26)]
    start_urls = ["https://www.olx.ua/nedvizhimost/kvartiry/?currency=USD"]
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        chrome_options = Options()
        chrome_options.add_argument("--headless")
        chrome_options.add_argument("--disable-gpu")
        chrome_options.add_argument("--no-sandbox")
        self.driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=chrome_options)

    def start_requests(self):
        try:
            for url in self.start_urls:
                self.driver.get(url)
                self.driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
                self.driver.implicitly_wait(5)
                body = self.driver.page_source
                response = HtmlResponse(url=self.driver.current_url, body=body, encoding='utf-8')


                yield from self.parse(response)
        finally:
            self.driver.quit()  # Обеспечиваем закрытие браузера
    def parse(self, response):
        item = OlxItem()
        item["href"] = response.css('a.css-1tqlkj0::attr(href)').getall()
        yield item
````
### Item
````
import scrapy


class OlxItem(scrapy.Item):
    # define the fields for your item here like:
    href = scrapy.Field()
````
### Pipeline
````
import json

from itemadapter import ItemAdapter


class OlxPipeline:
    def process_item(self, item, spider):
        l = len(item['href'])
        for i in range(l):
            item['href'][i] = "https:/" + item['href'][i]
        with open("product.json", "a", encoding="utf-8") as f:
            json.dump(dict(item), f, ensure_ascii=False, indent=4)
        return item
````
### Settings
````
ITEM_PIPELINES = {
   "olx.pipelines.OlxPipeline": 300,
}
````
### Основные настройки
````
BOT_NAME – имя вашего паука. Scrapy использует его для идентификации проекта.

SPIDER_MODULES – список модулей, в которых находятся пауки.

NEWSPIDER_MODULE – модуль, в котором создаются новые пауки.

Настройки User-Agent и соблюдение правил
USER_AGENT – задает строку User-Agent, используемую при отправке запросов. В этом случае используется библиотека Faker, чтобы генерировать случайный User-Agent.

ROBOTSTXT_OBEY – если True, Scrapy будет соблюдать правила, прописанные в robots.txt сайтов.

Параметры для управления запросами
CONCURRENT_REQUESTS – максимальное количество запросов, выполняемых одновременно (по умолчанию 16).

DOWNLOAD_DELAY – задержка между запросами (например, 3 означает 3 секунды между запросами).

CONCURRENT_REQUESTS_PER_DOMAIN – ограничение количества запросов к одному домену.

CONCURRENT_REQUESTS_PER_IP – ограничение количества запросов к одному IP-адресу.

Cookies и Telnet Console
COOKIES_ENABLED – если False, Scrapy не будет сохранять cookies.

TELNETCONSOLE_ENABLED – включение/отключение Telnet-консоли.

Настройки заголовков HTTP-запроса
DEFAULT_REQUEST_HEADERS – переопределение заголовков по умолчанию (например, Accept-Language).

Настройки middlewares
SPIDER_MIDDLEWARES – middlewares, обрабатывающие данные внутри паука.

DOWNLOADER_MIDDLEWARES – middlewares, отвечающие за загрузку данных.

Настройки расширений
EXTENSIONS – активация/деактивация Scrapy-расширений.

Настройки pipelines
ITEM_PIPELINES – задает обработчики (pipelines) для обработки item. Значение 300 – приоритет исполнения.

Настройки авто-дросселирования (AutoThrottle)
AUTOTHROTTLE_ENABLED – если True, Scrapy автоматически регулирует скорость запросов.

AUTOTHROTTLE_START_DELAY – начальная задержка загрузки.

AUTOTHROTTLE_MAX_DELAY – максимальная задержка загрузки.

AUTOTHROTTLE_TARGET_CONCURRENCY – целевая степень параллельности запросов.

AUTOTHROTTLE_DEBUG – включение режима отладки AutoThrottle.

Настройки HTTP-кэширования
HTTPCACHE_ENABLED – если True, включается кэширование ответов.

HTTPCACHE_EXPIRATION_SECS – время жизни кэша (0 = бесконечно).

HTTPCACHE_DIR – папка для хранения кэша.

HTTPCACHE_IGNORE_HTTP_CODES – список HTTP-кодов, которые не должны кэшироваться.

HTTPCACHE_STORAGE – тип хранилища кэша (например, файловое).

Дополнительные настройки
FEED_EXPORT_ENCODING – кодировка при экспорте данных (utf-8, чтобы сохранять JSON и CSV корректно).
````
### Scarping API
````
import scrapy
import json

class APIParserSpider(scrapy.Spider):
    name = "api_parser"
    
    def start_requests(self):
        url = "https://jsonplaceholder.typicode.com/posts"
        yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        data = json.loads(response.text)  # Декодируем JSON-ответ
        for item in data:
            yield {
                "id": item["id"],
                "title": item["title"],
                "body": item["body"]
            }
````
````
yield scrapy.Request(url=url, headers={"Authorization": "Bearer YOUR_API_KEY"}, callback=self.parse)
````