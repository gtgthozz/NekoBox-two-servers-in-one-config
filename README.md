# NekoBox-two-servers-in-one-config
**Languages**: [![ru](https://img.shields.io/badge/lang-ru-red.svg)](https://github.com/gtgthozz/NekoBox-two-servers-in-one-config/blob/main/README-ru.md)

Using 2 (or more) servers in the same configuration with different routing rules. It will be useful if you have 2 servers, but you don't want to switch between them all the time. 

The previous approach turned out to have far more issues than I expected. So I found two more options.

*GUI - Graphical User Interface

**Option 1:**

1.1 Download this fork — https://github.com/Mahdi-zarei/nekoray/releases/  
1.2 Create a routing rule with the necessary attribute in the settings.  
1.3 Set the required config as `outbound`. Done!

**Pros**: Everything is done in the GUI. The routing rules have been significantly improved. The program now includes DNS Hijack (I haven't used it yet, but it sounds like a cool feature).  
**Cons**: Configs from the original NekoRay don’t seem to be supported (it didn’t like something in my TLS settings). There are some bugs (I couldn’t load geo-databases through a link). The original NekoRay has been tested over time, but this fork hasn’t. There might be something "undesirable" that comes up.

**Option 2:**

2.1 Export both configs (let’s call them "config A" and "config B") in sing-box format (right-click -> share -> export to sing-box). If you export as "Core config" instead of "Test config," more settings will be exported. It makes sense to export the first config as "Core," while the second can be exported as "Test."

2.2 Then, you need to combine them in a text editor (I used VSC). In the merged config, insert `outbounds` and `route` from config A. Then add the server information from `outbounds` in config B to the `outbounds` in config A.

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

2.4 Now, you’ll have 2 servers in `outbounds`, and you can specify rules to be used instead of those set in the GUI. However, I prefer to set rules in the GUI.

2.5 Copy the resulting `outbounds` configuration (and save it somewhere as a backup) → Open Neko → Double-click the desired profile (I’ll paste it into profile A) → Additional config settings → Click the button (by logic, it should have a text like “not set”) → Paste the resulting configuration. I only pasted the `outbounds` (in the example above, they are already inside the brackets `{}` that are already there by default).

2.6.1 You could also add routing settings (`{"route": {"rules": [...]}}`) into the additional config settings, but then the `rules` from the GUI for this config would not apply.

So: Settings → Routing settings → Basic routes → Custom presets. I used `domain_suffix`. The full list of what you can specify is here: https://sing-box.sagernet.org/configuration/route/rule/.

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

2.6.2 However, the GUI option has a downside. If you select a config that doesn’t have an `outbound` with the tag `PROXY-2`, Neko will throw an error: "create service: `outbound not found for rule [2]: PROXY-2`." It’s saying that `PROXY-2` is in the rules, but not in the config. So, you’ll either have to delete the `PROXY-2` rule from the custom presets each time, or keep `"route": {"rules": [...]}` from config A in the merged config-frankenstein. That way, you’ll set all routing rules inside the custom preset of the merged config, rather than in the GUI.

Just to mention, `"final": "bypass"` in `rules` is the "Default outbound" from the basic route settings. I would remove `"final"`, `"auto_detect_interface"`, and `"geoip":` with `"geosite":` from the `"route"`. If they are not set, Neko will change them depending on GUI settings.

**Pros**: Compared to adding a sing-box config I manually created last time (not the first option), traffic counting works in the GUI here, and switching between TUN mode and system proxy mode now works.  
**Cons**: Compromises with routing settings. The GUI doesn’t natively support this, so it’s a workaround.

**Bonus**: The Antizapret geo-database loaded from GitHub (sometimes gives a timeout and doesn’t let the profile start :/). Need beta V4 and higher.

This chunk is inserted into the Additional config settings:

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
} (this may seem unnecessary, but it’s inserted after outbounds)
```

And this is in my basic routes (again, I’m using `rules` from the GUI):
```json
{
    "outbound": "proxy",
    "rule_set": "antizapret"
},
```

### [Example config and base routes](https://github.com/gtgthozz/NekoBox-two-servers-in-one-config/blob/main/Example%20of%20additional%20config%20settings%20and%20basic%20routes%20for%202%20proxies%20in%20one%20profile%20%2B%20a%20rule%20set%20with%20an%20Antizapret%20database%20(rule_set).json)

---
Mirror: [![en](https://img.shields.io/badge/Google_Docs-en-white.svg)](https://docs.google.com/document/d/e/2PACX-1vT4MMzIPlGMxdVmvySx4C-QZKT9EO5XQAISCwZAT0zQznfqmYDlzjnJomhzaDTPrZMKhGWIjQWZwcyc/pub)
