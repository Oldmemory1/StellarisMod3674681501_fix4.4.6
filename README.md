# 塞塔娜直接宣战

Stellaris「机械纪元 / The Machine Age」合成女王（Cetana）危机的改动 mod：
允许玩家（先辈子弟 / Scion）在 `scion_overlord_warning_attend.1` 事件中选择「宣战」，
直接对合成女王开战，并可召集堕落帝国并肩作战。

- 支持版本：**v4.4.6**
- 类型：Balance

---

## 修改记录（Bug 修复）

### 1. 修复：战胜 Cetana 后所有堕落帝国无法正常外交

- **现象**：选择「宣战」并击败合成女王后，无法再对任何堕落 / 觉醒堕落帝国进行外交
  （宣战、提议交易、设为对手、侮辱、索取宣称等均不可用）。
- **根因**：原版危机开始事件 `crisis.8005` 会设置全局旗标 `synth_queen_happened`，
  且**原版全程从不移除**。`common/diplomatic_actions/00_actions.txt` 中有 5 个针对堕落帝国的
  外交行动以此旗标为门槛（失败提示 `the_fe_is_busy_with_synth_queen`），因此女王一出现，
  对堕落的外交就被永久禁用（此为**原版设计**）。本 mod 让玩家能战胜 Cetana，但该旗标依旧存在，
  于是堕落外交永久卡死。
- **修复**：新增 `common/diplomatic_actions/zzz_synth_queen_fe_diplomacy_fix.txt`，
  按 key 覆盖以下 5 个行动，并在门槛中追加放行条件 `has_global_flag = synth_queen_defeated`——
  即**击败 Cetana 之后**（原版 `crisis.23015` 会设置该旗标）自动恢复对堕落的外交：
  - `action_declare_war`（宣战）
  - `action_offer_trade_deal`（提议交易 / 协议）
  - `action_make_rival`（设为对手）
  - `action_insult`（侮辱）
  - `action_make_claims_diplomacy_view`（索取宣称）
- **保留** `synth_queen_happened` 不动，因此不影响：危机再触发保护（`crisis_trigger_events.txt`）、
  以及「女王已被击败」的复合判定（成就、抱负、银河焦点等战后内容）。
- **说明**：覆盖文件名刻意区别于原版 `00_actions.txt`（同名会整文件替换、丢失其它所有行动），
  仅按 key 覆盖上述 5 个行动，其余原版外交行动不受影响。

### 2. 修复：战争目标字段拼写错误

- **文件**：`common/war_goals/999_00_war_goals.txt`，战争目标 `synth_queen_wg_end_threat`。
- **问题**：`hide_if_no_cb = yes` 不是合法字段（原版 common/ 中出现 0 次）。
- **修复**：改为原版同类战争目标 `wg_end_threat` 所用的正确字段 **`hide = no_cb`**。

### 3. 修复：crisis.8042 中 country_event 作用域错误

- **文件**：`events/0991_machine_age_crisis_events.txt`，事件 `crisis.8042`（由 `crisis.8030` 调度触发）。
- **问题**：`country_event = { id = crisis.8042 }` 被误写在 `every_system_within_border`
  （**星系作用域**）内部，而 `country_event` 需要**国家作用域**。
- **修复**：参照原版 `crisis.8042`，将该 `country_event` 移出 `every_system_within_border`、
  置于国家作用域（`if` 步骤一分支内），使「重新触发」逻辑变为每国一次。
  保留 mod 自己的 `random = 50` 取值及其余特殊效果不变。

### 4. 修复：本地化键 `cetana_desc` 在中文中未定义

- **文件**：`localisation/simp_chinese/sqw_machine_age_l_simp_chinese.yml`。
- **问题**：`synth_queen_spawn`、`crisis.23005` 等在创建塞塔娜头目时使用
  `custom_description = cetana_desc`，但**原版从未在任何本地化文件中定义 `cetana_desc` 键**，
  因此该键在中文下未定义（linter 报错，游戏内会显示原始键名）。
- **修复**：在 mod 中文本地化中补上 `cetana_desc` 键，文本参照原版塞塔娜描述
  （`resolution_crisis_analysis_cetana_desc`）。

### 5. 修复：`synth_queen_fe_war` 中空的 `else_if` 块（CWTools 警告）

- **文件**：`common/scripted_effects/0992_machine_age_effects.txt`，效果 `synth_queen_fe_war`。
- **问题**：处理「玩家在联邦中但非盟主」的 `else_if` 分支只有 `limit`、没有任何效果（空块），
  CWTools 报「空 if 块」警告。
- **修复**：该分支本就是无操作（此情形无法接管他人联邦），删除这个空 `else_if`；
  行为不变——直接落到下方宣战块，由堕落 / 玩家各自对塞塔娜宣战（等同原版无联邦分支的行为）。

### 6. 修复：`NOT` 多子条件语义错误（应为 `NOR`）

- **文件**：`events/0991_machine_age_crisis_events.txt`，事件 `crisis.30000`（每 180 天替换非天灾模板空间站）。
- **问题**：删除女王空间站的 `limit` 中，排除女王两种模板星垒用了
  `NOT = { is_ship_size = big_starbase_synth_queen  is_ship_size = starbase_synth_queen }`。
  Paradox 脚本中 `NOT` 带多个子条件按 **NAND**（非「全部成立」）求值；而单艘船只能有一个
  `is_ship_size`，两条永不同时成立，故该 `NOT` **恒为真**，排除形同虚设——
  女王自己的模板星垒也会被误删。
- **修复**：改为原版负逻辑惯用写法 **`NOR = { ... }`**（两者皆不成立才为真），
  使女王的 `big_starbase_synth_queen` 与 `starbase_synth_queen` 两种模板星垒都被正确排除、不再误删。

---

## 开发说明

项目用 `uv` 管理 Python 环境（用于脚本化校验，如 Paradox 脚本花括号配平检查）：

```bash
uv run python <script>
```
