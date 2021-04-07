# CloudEvents



| 字段 | 含义 | 是否必须 | Http Header key |
| :--- | :--- | :--- | :--- |
| id | 事件 id | 是 | Ce-Id |
| specversion | 使用的 cloudEvents 规范版本 | 是 | Ce-Specversion |
| type | 发送方定义的事件类型 | 是 | Ce-Type |
| source | 事件来源，包含 ns 与事件源名称 | 是 | Ce-Source |
| subject | 事件的主题 | 可选 | Ce-Subject |
| time | 事件产生的时间 | 可选 | Ce-Time |
| datacontenttype | Data 数据格式 | 可选 | Content-Type |
| data | 消息数据 | 可选 | Http body 中 |

