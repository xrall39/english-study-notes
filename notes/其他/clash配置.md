## Clash 配置

使用教程两步
1. 将以下**预处理文件配置**yaml文件复制进自己的订阅配置的**配置文件预处理**选项中(配置右键即可添加)
2. 更新配置即可

该配置规则是基于 Github中 [clash-rules](https://github.com/Loyalsoldier/clash-rules) 进行配置。

配置说明，该黑名单文字描述与功能无效，因在 clash-rules 中，**prepend-rules** 黑名单配置不一样，如需切换黑名单，需要替换 **prepend-rules** 下的规则达到黑名单的文字描述。

白名单模式，意为「没有命中规则的网络流量，统统使用代理」，适用于服务器线路网络质量稳定、快速，不缺服务器流量的用户。

```yaml
## 白名单规则
rules:
  - RULE-SET,applications,DIRECT
  - DOMAIN,clash.razord.top,DIRECT
  - DOMAIN,yacd.haishan.me,DIRECT
  - RULE-SET,private,DIRECT
  - RULE-SET,reject,REJECT
  - RULE-SET,icloud,DIRECT
  - RULE-SET,apple,DIRECT
  - RULE-SET,google,PROXY
  - RULE-SET,proxy,PROXY
  - RULE-SET,direct,DIRECT
  - RULE-SET,lancidr,DIRECT
  - RULE-SET,cncidr,DIRECT
  - RULE-SET,telegramcidr,PROXY
  - GEOIP,LAN,DIRECT
  - GEOIP,CN,DIRECT
  - MATCH,PROXY
```

黑名单模式，意为「只有命中规则的网络流量，才使用代理」，适用于服务器线路网络质量不稳定或不够快，或服务器流量紧缺的用户。通常也是软路由用户、家庭网关用户的常用模式。

```yaml
## 黑名单规则
rules:
  - RULE-SET,applications,DIRECT
  - DOMAIN,clash.razord.top,DIRECT
  - DOMAIN,yacd.haishan.me,DIRECT
  - RULE-SET,private,DIRECT
  - RULE-SET,reject,REJECT
  - RULE-SET,tld-not-cn,PROXY
  - RULE-SET,gfw,PROXY
  - RULE-SET,telegramcidr,PROXY
  - MATCH,DIRECT
```

预处文件配置

```yaml
## 预处理文件配置
parsers:
  - reg: ^.*$
    code: |
      module.exports.parse = (raw, { yaml }) => {
        const rawObj = yaml.parse(raw)
        const groups = []
        const rules = []
        return yaml.stringify({ ...rawObj, 'proxy-groups': groups, rules })
      }
    yaml:
      prepend-proxy-groups: # 建立策略组
        - name: 🔯 代理模式
          type: select
          proxies:
            - 绕过大陆丨黑名单(GFWlist) # 黑名单模式，意为「只有命中规则的网络流量，才使用代理」
            - 绕过大陆丨白名单(Whitelist) # 白名单模式，意为「没有命中规则的网络流量，统统使用代理」

        - name: 🔰 选择节点
          type: select

        - name: 🛑 广告拦截
          type: select
          proxies:
            - DIRECT
            - REJECT
            - PROXY

        - name: 绕过大陆丨黑名单(GFWlist)
          type: url-test
          url: http://www.gstatic.com/generate_204
          interval: 86400
          proxies:
            - DIRECT

        - name: 绕过大陆丨白名单(Whitelist)
          type: url-test
          url: http://www.gstatic.com/generate_204
          interval: 86400
          proxies:
            - PROXY

        - name: PROXY
          type: url-test
          url: http://www.gstatic.com/generate_204
          interval: 86400
          proxies:
            - 🔰 选择节点
      commands:
        - proxy-groups.🔰 选择节点.proxies=[]proxyNames # 向指定策略组添加订阅中的节点名，可使用正则过滤
        - proxy-groups.🔰 选择节点.proxies.0+DIRECT # 向指定分组第一个位置添加一个 DIRECT 节点名
      prepend-rules:
        - RULE-SET,applications,DIRECT
        - DOMAIN,clash.razord.top,DIRECT
        - DOMAIN,yacd.haishan.me,DIRECT
        - RULE-SET,private,DIRECT
        - RULE-SET,reject,REJECT
        - RULE-SET,icloud,DIRECT
        - RULE-SET,apple,DIRECT
        - RULE-SET,google,DIRECT
        - RULE-SET,proxy,PROXY
        - RULE-SET,direct,DIRECT
        - RULE-SET,lancidr,DIRECT
        - RULE-SET,cncidr,DIRECT
        - RULE-SET,telegramcidr,PROXY
        - GEOIP,LAN,DIRECT
        - GEOIP,CN,DIRECT
        - MATCH,PROXY
      mix-rule-providers:
        reject:
          type: http
          behavior: domain
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt"
          path: ./ruleset/reject.yaml
          interval: 86400

        icloud:
          type: http
          behavior: domain
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/icloud.txt"
          path: ./ruleset/icloud.yaml
          interval: 86400

        apple:
          type: http
          behavior: domain
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/apple.txt"
          path: ./ruleset/apple.yaml
          interval: 86400

        google:
          type: http
          behavior: domain
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/google.txt"
          path: ./ruleset/google.yaml
          interval: 86400

        proxy:
          type: http
          behavior: domain
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/proxy.txt"
          path: ./ruleset/proxy.yaml
          interval: 86400

        direct:
          type: http
          behavior: domain
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/direct.txt"
          path: ./ruleset/direct.yaml
          interval: 86400

        private:
          type: http
          behavior: domain
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/private.txt"
          path: ./ruleset/private.yaml
          interval: 86400

        gfw:
          type: http
          behavior: domain
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/gfw.txt"
          path: ./ruleset/gfw.yaml
          interval: 86400

        tld-not-cn:
          type: http
          behavior: domain
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/tld-not-cn.txt"
          path: ./ruleset/tld-not-cn.yaml
          interval: 86400

        telegramcidr:
          type: http
          behavior: ipcidr
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/telegramcidr.txt"
          path: ./ruleset/telegramcidr.yaml
          interval: 86400

        cncidr:
          type: http
          behavior: ipcidr
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/cncidr.txt"
          path: ./ruleset/cncidr.yaml
          interval: 86400

        lancidr:
          type: http
          behavior: ipcidr
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/lancidr.txt"
          path: ./ruleset/lancidr.yaml
          interval: 86400

        applications:
          type: http
          behavior: classical
          url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/applications.txt"
          path: ./ruleset/applications.yaml
          interval: 86400

```