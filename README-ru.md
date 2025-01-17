# NekoBox-two-servers-in-one-config
**Languages**: [![en](https://img.shields.io/badge/lang-en-red.svg)](https://github.com/gtgthozz/NekoBox-two-servers-in-one-config/blob/main/README.md)

Использование 2-х (или более) серверов в одном конфиге с разными правилами маршрутизации. Будет полезно, если у вас есть 2 сервера, но вы не хотите между ними постоянно переключаться. 

*GUI - Графический интерфейс

**Вариант 1:**

1.1 Скачать этот форк — https://github.com/Mahdi-zarei/nekoray/releases/

1.2 Создать в настройках маршрутов правило с нужным атрибутом.

1.3 В качестве `outbound` указать нужный конфиг. Всё!

**Плюсы**: всё делается в GUI. Правила маршрутизации заметно прокачались. В программе появился DNS Hijack (сам ещё не пользовался, но по описанию крутая фича). 

**Минусы**: вроде конфиги из оригинального неко не поддерживаются (У меня вроде ему что-то в TLC не нравилось). Есть какое-то количество багов (я вообще никак так и не смог подтянуть геобазы по ссылке). Оригинальный неко проверен временем, а этот форк ещё нет. Вдруг там обнаружится что-нибудь «нежелательное».

**Вариант 2:**

2.1 Экспортировать оба конфига (назовём их «конфиг А» и «конфиг Б») в sing-box формате (ПКМ —> поделиться —> экспорт в sing-box). Если экспортировать как «Core config», а не «Test config», будет экспортированно больше настроек. Первый конфиг есть смысл экспортировать как «Core», а второй уже можно как «Test».

2.2 Далее нужно их объединить в текстовом редакторе (я использовал VSC). В совмещенный вариант нужно вставить `outbounds` и `route` конфига А. Далее добавить в `outbounds` конфига А информацию о втором сервере из `outbounds` конфига Б.

```json
{
"outbounds": [
        {
            "domain_strategy": "",
            "flow": "xtls-rprx-vision",
            "packet_encoding": "",
            "server": "IP OR DOMAIN",
            "server_port": 443,
            "tag": "proxy",
            "tls": {
                "enabled": true,
                "reality": {
                    "enabled": true,
                    "public_key": "TEXT",
                    "short_id": "TEXT"
                },
                "server_name": "telegram.org",
                "utls": {
                    "enabled": true,
                    "fingerprint": "chrome"
                }
            },
            "type": "vless",
            "uuid": "66600rnd-5ddf-1234-a123-1234abscdg2"
        },
        {
            "domain_strategy": "",
            "flow": "",
            "packet_encoding": "",
            "server": "IP OR DOMAIN",
            "server_port": 8080,
            "tag": "PROXY-2",
            "tls": {
                "enabled": true,
                "reality": {
                    "enabled": true,
                    "public_key": "TEXT2",
                    "short_id": "TEXT2"
                },
                "server_name": "TEXT2",
                "utls": {
                    "enabled": true,
                    "fingerprint": "chrome"
                }
            },
            "type": "vless",
            "uuid": "77700rnd-1234-1234-a123-1234abscdg2"
        },
        {
            "tag": "direct",
            "type": "direct"
        },
        {
            "tag": "bypass",
            "type": "direct"
        },
        {
            "tag": "block",
            "type": "block"
        },
        {
            "tag": "dns-out",
            "type": "dns"
        }
    ]
}
```
(Далее можно было бы вписать в `route` из конфига А нужные `rules`, но о них позже.)```

2.4 Получилось, что в `outbounds` прописаны 2 сервера + далее можно прописать правила, которые будут использоваться вместо тех, что заданы в GUI. Но я хочу задавать правила в GUI. 

2.5 Далее нужно скопировать (а лучше ещё и где-нибудь сохранить) получившуюся настройку `outbounds` —> открыть неко —> нажать по нужному профилю два раза (я вставлю в профиль А) —> Доп. настройки конфига —> нажать на кнопку (по логике у неё должен быть текст «не задано») —> вставить получившиеся настройки. Я вставлю только `outbounds` (в примере выше они уже вставлены в скобочки {}, которые будут по умолчанию там написаны). 

2.6.1 Настройки маршрутов (`{"route": {"rules": [...]}}`) также можно было бы вписать в Доп. настройки конфига, но тогда `rules` из GUI для этого конфига не применялись бы.

Итак: Настройки —> Настройки маршрутов —> Базовые маршруты —> Кастомные прессеты. Я указал `domain_suffix`. Полный список всего, что там можно указать, тут: https://sing-box.sagernet.org/configuration/route/rule/. 

```json
{
    "rules": [
        {
            "domain_suffix": [
                "domain.name.com",
                "domain.name2.com"
            ],
            "outbound": "PROXY-2"
        }
    ]
}
```

2.6.2 Однако у варианта с GUI есть минус. Если вы выберите конфиг, где нет `outbounds` с тегом `PROXY-2`, неко выдаст ошибку create service: `outbound not found for rule [2]: PROXY-2`. Мол, у меня в правилах `PROXY-2` есть, а в конфиге его нет. И тут либо каждый раз удалять из кастомных прессетов правило с `PROXY-2`. Либо оставлять в сдвоенном конфиге-франкенштейне `"route": {"rules": [...]}` из конфига А. Вписывая все правила маршрутизации в кастомный прессет получившегося сдвоенного конфига, а не задавать их в GUI. 

На всякий случай скажу, что `"final": "bypass"` в `rules` — это «Outbound поумолчанию» из настроек Базовых маршрутов. Из `"route"` я бы удалил: `"final"`, `"auto_detect_interface"` и `"geoip":` с `"geosite":`. Если они не заданы, то неко в зависимости от настроек в GUI будет сам их менять. 

**Плюсы**: по сравнению с добавлением sing-box конфига, который я натыкал в прошлый раз (речь не о первом варианте), тут работает подсчёт трафика в GUI. И теперь работает переключение между TUN режимом и режимом системного прокси.
**Минусы**: компромисс с настройкой маршрутов. Всё-таки GUI неко изначально такое не поддерживает. Так что это костыль. 

**Бонус**: геобаза антизапрета, подгружаемая с гитхаба (у меня переодически выдаёт таймаут и не даёт запуститься профилю :/). Работает только на версиях от бетта 4 и выше.

Этот кусок вставляется в Доп. настройки конфига:

```json
], (closing bracket for outbounds)
"route": {
    "rule_set": [
        {
            "download_detour": "direct",
            "format": "binary",
            "tag": "antizapret",
            "type": "remote",
            "url": "https://github.com/savely-krasovsky/antizapret-sing-box/releases/latest/download/antizapret.srs"
        }
    ]
}
} (может оказаться лишней, если вставить после outbounds, не удалив закрывающую скобку)
```


Базовые маршруты. (опять таки, так как я `rules` беру из GUI):
```json
{
    "outbound": "proxy",
    "rule_set": "antizapret"
},
```

### [Пример конфига и базовых маршрутов](https://github.com/gtgthozz/NekoBox-two-servers-in-one-config/blob/main/Example%20of%20additional%20config%20settings%20and%20basic%20routes%20for%202%20proxies%20in%20one%20profile%20%2B%20a%20rule%20set%20with%20an%20Antizapret%20database%20(rule_set).json)

---
Mirror: [![ru](https://img.shields.io/badge/Google_Docs-ru-white.svg)](https://docs.google.com/document/d/e/2PACX-1vSGGOEIlE-jXbRenm_tiZg7-6oizadGQ8tjbPx4ozb2gg0sSsRkivzunSF1xnZpBEiwZJtkzRGKhLSu/pub)
