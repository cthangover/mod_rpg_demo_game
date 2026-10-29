# RPG Demo — 一个小小的图块世界

RPG Demo 是一段短小而完整的图块地图冒险：灰森林旁的一座村庄、村长的家，以及洞穴里住着一只狼的森林。你用键盘行走、与人们交谈、开宝箱、在森林里采集，再把东西卖给商人，然后战斗。它只由数据文件和剧本构成——没有一行代码——因此同时也是“一款图块地图游戏是如何组装起来的”的现成示例。

这个演示没有自己的游戏卡片：它是 **RPG Plus** 游戏的世界，RPG Plus 包含本模组并在其之上扩建。从“开始游戏 → RPG Plus → 新游戏”进入：走完 RPG Plus 简短的岔路口序章后，你就会抵达这座村庄。下面的截图来自 RPG Plus，所以其中一些人和商品是那个游戏加上的。

![夜晚的灰森林旁村庄：房屋、亮着灯的石板广场、村民、池塘，以及小路上的英雄。](assets/village.jpg)

## 玩法

- 用 WASD 或方向键**行走**，按 Shift **奔跑**。
- 面对人物或物件时按 E 或 F **交谈、打开、阅读**。
- 走到门口或路的边缘，就会被传送到下一张地图。

剧情：村长请你赶走盘踞在村庄东边洞穴旁的狼。经过池塘，沿着林间小路，解决挡路的史莱姆，然后与狼正面对决。狼被击败后，村长会谢谢你——用他自己的方式。

## 世界里有什么

- **三张地图。** 村庄（32×18 图块）、村长的家和灰森林，全部用 Village Pack 的村庄图块集绘制。
- **人物。** 村长发放并完结任务，村民闲聊，一块路牌写着村庄的名字；有一扇门锁着。
- **宝箱。** 村长家里一个、森林里一个；打开后保持敞开。
- **森林采集。** 小路沿线散落着柴火、石头和森林蘑菇；每个采集点只能采一次。一朵蘑菇恢复 5 点生命。
- **商人。** 以 10 枚硬币出售药剂（恢复 20 点生命），并以 1 枚收购柴火、2 枚收购石头。
- **战斗。** 挡路的史莱姆和洞穴旁的狼都显示在地图上，与之交谈就会开战。在森林里，大约每走 48 步会遇到一次随机战斗，对手是一到两只史莱姆。

![灰森林：通向狼所在洞穴的小路，一个宝箱、石头、蘑菇，以及挡路的史莱姆。](assets/forest.jpg)

![商人的店：左边是商品，中间是选中的药剂及其价格和购买按钮，右边是你的背包。RPG Plus 给这家店添加了更多商品。](assets/shop.jpg)

![一场与史莱姆的战斗：英雄在左下角，史莱姆在上方，战斗记录在右侧。](assets/battle.jpg)

## 给模组作者：它是怎么搭起来的

```filestree
rpg_demo_game/
├── maps/                     rpg_village.json、rpg_elder_house.json、rpg_forest.json
├── map_events/<map>/events.json   人物、门、宝箱、可拾取物、怪物
├── scenes/                   每张地图一个场景
├── scenarios/rpg_demo/       开场、地图入口和 events/*.scenario
├── characters/               英雄、史莱姆和狼
├── items/rpg_demo/           柴火、石头、蘑菇、药剂及其图标
├── shops/village_merchant.json
└── locale/                   七种语言的名称和剧本翻译
```

**地图**是叠在图块集上的多层图块编号网格，带有自己的名称、尺寸和缩放。森林地图还列出了它的随机遭遇：

```json
"Encounters": {
  "Steps": 48,
  "Groups": ["rpg_slime", "rpg_slime,rpg_slime"]
}
```

**事件**是放在图块上的东西。带 `"Trigger": "Touch"` 的门负责传送；怪物会运行一段剧本，并在自己的标志位被设置后消失：

```json
{
  "Id": "wolf_boss",
  "X": 25,
  "Y": 8,
  "Sprite": "village/wolf",
  "Dir": "Left",
  "Scenario": "rpg_demo/events/wolf_boss",
  "Condition": "rpg_wolf_dead != 1"
}
```

**剧本**说出台词并开始战斗；获胜后设置标志位：

```scenario
ptext "一只狼坐在洞穴旁。它打量你的眼神，就像打量村长的山羊。" key=rpg_demo/wolf/line1
text "嗷呜——" key=rpg_demo/wolf/line2
action rpg.battle enemies=rpg_wolf,rpg_slime win_flag=rpg_wolf_dead
end
```

本模组剧本中的台词以俄语写成；每行都有一个 `key=`，对应的翻译放在 `locale/scenarios_<lang>.properties` 中，所以游戏会按界面语言显示它们：

```properties
rpg_demo/wolf/line1 = 一只狼坐在洞穴旁。它打量你的眼神，就像打量村长的山羊。
```

其余部分都是同一个套路：宝箱执行 `action item.add`、设置标志位，然后被一个带相反 `Condition` 的“已开宝箱”事件替换；商人执行 `action trade.open shop=rpg_demo/village_merchant`；首次进入村庄会运行开场剧本，用 `action party.add id=rpg_hero` 把英雄放进队伍。

想以可视化方式修改地图和事件，请使用 RPG 编辑器工具中的地图编辑器。

## 出问题了怎么办

1. **门没有反应。** `Touch` 门在你踩上去时触发；`Action` 门需要面向它按 E 或 F。
2. **怪物或宝箱复活了。** 它的 `Condition` 必须检查剧本所设置的同一个标志位。
3. **商店是空的。** 游戏中必须包含 trading 模组，且剧本里的商店 id 必须与 `shops/village_merchant.json` 一致。
