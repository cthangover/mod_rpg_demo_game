# RPG Demo — một thế giới tile-map nhỏ

RPG Demo là một cuộc phiêu lưu tile-map ngắn gọn và trọn vẹn: một ngôi làng bên Rừng Xám, nhà của trưởng làng, và khu rừng có con sói trú ở bên hang. Bạn đi lại bằng bàn phím, nói chuyện với mọi người, mở rương, hái lượm trong rừng, bán cho thương nhân rồi chiến đấu. Nó chỉ gồm các tệp dữ liệu và kịch bản — không một dòng mã — nên đồng thời cũng là một ví dụ sống động về cách một trò chơi tile-map được ghép lại.

Bản demo không có thẻ trò chơi riêng: nó là thế giới của trò **RPG Plus**, trò chơi này chứa mod và xây tiếp trên nền nó. Bắt đầu từ **Chơi → RPG Plus → Trò chơi mới**: đi qua phần mở đầu ngắn ở ngã tư của RPG Plus là bạn đến ngôi làng này. Các ảnh chụp bên dưới lấy từ RPG Plus, nên một số người và món hàng nhìn thấy là do trò đó thêm vào.

![Ngôi làng bên Rừng Xám về đêm: những ngôi nhà, quảng trường lát đá có đèn, dân làng, cái ao và anh hùng trên con đường mòn.](assets/village.jpg)

## Cách chơi

- **Đi lại** bằng WASD hoặc phím mũi tên, **chạy** bằng Shift.
- **Nói chuyện, mở, đọc** bằng E hoặc F khi đang quay mặt về phía người hoặc vật.
- Bước lên một cánh cửa hoặc mép đường sẽ đưa bạn sang bản đồ kế tiếp.

Cốt truyện: trưởng làng nhờ bạn đuổi con sói trú ở hang phía đông làng. Đi qua cái ao, men theo đường mòn trong rừng, giải quyết con slime chặn đường rồi đối mặt với con sói. Khi nó gục ngã, trưởng làng cảm ơn bạn — theo kiểu của ông ta.

## Trong thế giới có gì

- **Ba bản đồ.** Ngôi làng (32×18 ô), nhà trưởng làng và Rừng Xám, tất cả vẽ bằng tileset ngôi làng của Village Pack.
- **Con người.** Trưởng làng giao và kết thúc nhiệm vụ, dân làng tám chuyện, tấm biển gọi tên ngôi làng; một cánh cửa bị khóa.
- **Rương.** Một cái trong nhà trưởng làng, một cái trong rừng; mở rồi thì vẫn mở.
- **Hái lượm trong rừng.** Dọc các đường mòn có củi khô, đá và nấm rừng; mỗi chỗ chỉ nhặt được một lần. Một cây nấm hồi 5 máu.
- **Thương nhân.** Bán Thuốc tiên (hồi 20 máu) giá 10 đồng, mua củi khô 1 đồng và đá 2 đồng.
- **Chiến đấu.** Con slime trên đường mòn và con sói bên hang hiện rõ trên bản đồ và bắt đầu trận đánh khi bạn nói chuyện với chúng. Trong rừng, những trận ngẫu nhiên với một hai con slime xảy ra khoảng sau mỗi 48 bước chân.

![Rừng Xám: đường mòn tới hang của con sói, một cái rương, đá, nấm và con slime chắn đường.](assets/forest.jpg)

![Cửa hàng của thương nhân: hàng hóa bên trái, Thuốc tiên đang chọn kèm giá và nút mua, túi của bạn bên phải. RPG Plus thêm nhiều hàng hơn vào cửa hàng này.](assets/shop.jpg)

![Một trận đánh với slime: anh hùng ở dưới bên trái, slime ở phía trên, nhật ký chiến đấu bên phải.](assets/battle.jpg)

## Dành cho tác giả: nó được dựng thế nào

```filestree
rpg_demo_game/
├── maps/                     rpg_village.json, rpg_elder_house.json, rpg_forest.json
├── map_events/<map>/events.json   con người, cửa, rương, vật nhặt được, quái vật
├── scenes/                   mỗi bản đồ một cảnh
├── scenarios/rpg_demo/       phần mở đầu, lối vào bản đồ và events/*.scenario
├── characters/               anh hùng, slime và sói
├── items/rpg_demo/           củi khô, đá, nấm, thuốc tiên và biểu tượng của chúng
├── shops/village_merchant.json
└── locale/                   tên và bản dịch kịch bản bằng bảy ngôn ngữ
```

Một **bản đồ** là lưới số ô xếp thành các lớp trên một tileset, kèm tên, kích thước và mức thu phóng riêng. Bản đồ rừng còn liệt kê các trận ngẫu nhiên:

```json
"Encounters": {
  "Steps": 48,
  "Groups": ["rpg_slime", "rpg_slime,rpg_slime"]
}
```

**Sự kiện** là một thứ được đặt lên một ô. Cửa có `"Trigger": "Touch"` thì dịch chuyển; quái vật chạy một kịch bản và biến mất khi cờ của nó được bật:

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

**Kịch bản** nói các câu và bắt đầu trận đánh; thắng thì bật cờ:

```scenario
ptext "Một con sói ngồi bên hang. Nó nhìn bạn từ đầu đến chân như nhìn đàn dê của trưởng làng." key=rpg_demo/wolf/line1
text "Grừ." key=rpg_demo/wolf/line2
action rpg.battle enemies=rpg_wolf,rpg_slime win_flag=rpg_wolf_dead
end
```

Các câu trong kịch bản của mod này viết bằng tiếng Nga; mỗi câu có một `key=`, bản dịch nằm trong `locale/scenarios_<lang>.properties`, nên trò chơi hiện chúng theo ngôn ngữ giao diện của bạn:

```properties
rpg_demo/wolf/line1 = Một con sói ngồi bên hang. Nó nhìn bạn từ đầu đến chân như nhìn đàn dê của trưởng làng.
```

Các mảnh khác cũng hoạt động y như vậy: cái rương chạy `action item.add`, bật cờ rồi bị thay bằng sự kiện rương mở với `Condition` ngược lại; thương nhân chạy `action trade.open shop=rpg_demo/village_merchant`; lần đầu ghé làng sẽ chạy kịch bản mở đầu, đưa anh hùng vào đội bằng `action party.add id=rpg_hero`.

Muốn sửa bản đồ và sự kiện bằng hình ảnh trực quan, hãy dùng trình soạn bản đồ trong bộ công cụ soạn thảo RPG.

## Nếu có trục trặc

1. **Cửa không phản ứng.** Cửa kiểu `Touch` phát hiệu khi bạn giẫm lên; cửa kiểu `Action` cần E hoặc F khi đang đối mặt.
2. **Quái hay rương quay lại.** `Condition` của nó phải kiểm tra đúng lá cờ mà kịch bản bật.
3. **Cửa hàng trống rỗng.** Mod trading phải có trong trò chơi, và id cửa hàng trong kịch bản phải khớp `shops/village_merchant.json`.
