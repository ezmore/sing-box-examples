使用 **warp-reg**，注册warp

```
curl -sLo warp-reg https://github.com/badafans/warp-reg/releases/download/v1.0/main-linux-amd64 && chmod +x warp-reg && ./warp-reg
```

~使用 **warp-go**，注册warp~

```
curl -Lso- https://gitlab.com/ProjectWARP/warp-go/-/releases/v1.0.8/downloads/warp-go_1.0.8_linux_amd64.tar.gz | tar -zxf- warp-go && chmod +x warp-go && ./warp-go -register > /dev/null && ./warp-go -export-singbox wg.json > /dev/null && grep -Eo --color=never '"2606:4700:[0-9a-f:]+/128"|"private_key":"[0-9a-zA-Z\/+]+="|"reserved":\[[0-9]+(,[0-9]+){2}\]' wg.json && rm warp-go warp.conf wg.json
```

- 复制输出的 IPv6 地址，替换下面配置中的 `2606:4700::`
- 复制输出的 `private_key` 值，粘贴到下面配置中 `private_key` 后的 `""` 中
- 复制输出的 `reserved` 值，粘贴到下面配置中 `reserved` 后的 `[]` 中

**"outbounds"**
```jsonc
        {
            "type": "wireguard",
            "tag": "wireguard-out",
            "server": "162.159.192.1",
            "server_port": 2408,
            "local_address": [
                "172.16.0.2/32",
                "2606:4700::/128" // 粘贴你获得的 warp IPv6 地址，结尾加 /128
            ],
            "private_key": "", // 粘贴你的 "private_key" 值
            "peer_public_key": "bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=",
            "reserved":[0, 0, 0], // 粘贴你的 "reserved" 值
            "mtu": 1280
        }
```

编辑 **/root/sing-box_config.json**，按需增加 **"route"**，**"inbounds"**，**"outbounds"** 的内容（注意检查json格式），输入 `systemctl restart sing-box` 重启sing-box，访问bgp.he.net查看是否为Cloudflare的IP

**"route"**
```
        "rules": [
            {
                "protocol": [
                    "quic"
                ],
                "outbound": "block"
            },
            {
                "domain_keyword": [
                    "bgp.he.net"
                ],
                "geosite": [
                    "openai"
                ],
                "outbound": "wireguard-out"
            }
        ]
```

**"inbounds"**
```
            "sniff": true,
            "sniff_override_destination": true,
            "domain_strategy": "prefer_ipv6",
```

**配置示例**

```jsonc
{
    "log": {
        "level": "info",
        "timestamp": true
    },
    "route": {
        "rules": [
            {
                "protocol": [
                    "quic"
                ],
                "outbound": "block"
            },
            {
                "domain_keyword": [
                    "bgp.he.net"
                ],
                "geosite": [
                    "openai"
                ],
                "outbound": "wireguard-out"
            }
        ]
    },
    "inbounds": [
        {
            "sniff": true,
            "sniff_override_destination": true,
            "domain_strategy": "prefer_ipv6",
            // 粘贴你的服务端配置
        }
    ],
    "outbounds": [
        {
            "type": "direct",
            "tag": "direct"
        },
        {
            "type": "block",
            "tag": "block"
        },
        {
            "type": "wireguard",
            "tag": "wireguard-out",
            "server": "162.159.192.1",
            "server_port": 2408,
            "local_address": [
                "172.16.0.2/32",
                "/128" // 粘贴你获得的 warp IPv6 地址，结尾加 /128
            ],
            "private_key": "", // 粘贴你的 "private_key" 值
            "peer_public_key": "bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=",
            "reserved":[0, 0, 0], // 粘贴你的 "reserved" 值
            "mtu": 1280
        }
    ]
}
```
