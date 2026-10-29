# RPG Demo — un pequeño mundo de tiles

RPG Demo es una aventura corta y completa sobre un mapa de tiles: una aldea junto al Bosque Gris, la casa del anciano y el bosque, con un lobo viviendo junto a la cueva. Caminas con el teclado, hablas con la gente, abres cofres, recoges cosas en el bosque, se las vendes al mercader y luchas. Está hecho solo de archivos de datos y escenarios — ni una línea de código —, así que sirve de paso como ejemplo funcional de cómo se monta un juego de mapa de tiles.

La demo no tiene tarjeta de juego propia: es el mundo del juego **RPG Plus**, que incluye este mod y construye sobre él. Empiézalo desde **Jugar → RPG Plus → Nueva partida**: tras el breve prólogo del cruce de caminos de RPG Plus llegas a esta aldea. Las capturas de abajo vienen de RPG Plus, así que algunas personas y mercancías que se ven las añade ese juego.

![La aldea junto al Bosque Gris de noche: casas, una plaza adoquinada con farolas, aldeanos, un estanque y el héroe en el camino.](assets/village.jpg)

## Cómo se juega

- **Caminar** con WASD o las flechas, **correr** con Shift.
- **Hablar, abrir, leer** con E o F estando de frente a una persona o un objeto.
- Pisar una puerta o el borde del camino te lleva al siguiente mapa.

La historia: el anciano te pide echar al lobo que vive junto a la cueva al este de la aldea. Pasa el estanque, sigue el sendero del bosque, resuelve el limo que bloquea el paso y enfréntate al lobo. Cuando cae, el anciano te lo agradece — a su manera.

## Qué hay en el mundo

- **Tres mapas.** La aldea (32×18 tiles), la casa del anciano y el Bosque Gris, todos dibujados con el tileset de aldea de Village Pack.
- **Gente.** El anciano da y cierra la misión, los aldeanos charlan, un cartel nombra la aldea; una puerta está cerrada con llave.
- **Cofres.** Uno en la casa del anciano y otro en el bosque; una vez abiertos, siguen abiertos.
- **Recolectar en el bosque.** Por los senderos hay leña, piedras y setas del bosque; cada punto se recoge una sola vez. Una seta restaura 5 de salud.
- **El mercader.** Vende el Elixir (restaura 20 de salud) por 10 monedas y compra leña a 1 moneda y piedras a 2.
- **Combates.** El limo del sendero y el lobo de la cueva se ven en el mapa y empiezan una pelea cuando les hablas. En el bosque, encuentros aleatorios con uno o dos limos ocurren más o menos cada 48 pasos.

![El Bosque Gris: el sendero hacia la cueva del lobo, un cofre, piedras, setas y el limo en el camino.](assets/forest.jpg)

![La tienda del mercader: mercancías a la izquierda, el Elixir seleccionado con su precio y el botón de comprar, tu bolsa a la derecha. RPG Plus añade más mercancías a esta tienda.](assets/shop.jpg)

![Un combate contra un limo: el héroe abajo a la izquierda, el limo arriba, el registro de combate a la derecha.](assets/battle.jpg)

## Para autores: cómo está construido

```filestree
rpg_demo_game/
├── maps/                     rpg_village.json, rpg_elder_house.json, rpg_forest.json
├── map_events/<map>/events.json   personas, puertas, cofres, objetos recogibles, monstruos
├── scenes/                   una escena por mapa
├── scenarios/rpg_demo/       introducción, entradas de mapa y events/*.scenario
├── characters/               el héroe, el limo y el lobo
├── items/rpg_demo/           leña, piedras, seta, elixir y sus iconos
├── shops/village_merchant.json
└── locale/                   nombres y traducciones de escenarios en siete idiomas
```

Un **mapa** es una cuadrícula de números de tile en capas sobre un tileset, con su propio nombre, tamaño y zoom. El mapa del bosque también enumera sus encuentros aleatorios:

```json
"Encounters": {
  "Steps": 48,
  "Groups": ["rpg_slime", "rpg_slime,rpg_slime"]
}
```

Un **evento** es algo colocado sobre un tile. Una puerta con `"Trigger": "Touch"` teletransporta; un monstruo ejecuta un escenario y desaparece cuando su bandera queda activada:

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

El **escenario** dice las líneas y empieza la pelea; ganar activa la bandera:

```scenario
ptext "Un lobo está sentado junto a la cueva. Te mira igual que mira a las cabras del anciano." key=rpg_demo/wolf/line1
text "— Grrr." key=rpg_demo/wolf/line2
action rpg.battle enemies=rpg_wolf,rpg_slime win_flag=rpg_wolf_dead
end
```

Las líneas de los escenarios de este mod están escritas en ruso; cada línea lleva un `key=` con su traducción en `locale/scenarios_<lang>.properties`, así el juego las muestra en el idioma de tu interfaz:

```properties
rpg_demo/wolf/line1 = Un lobo está sentado junto a la cueva. Te mira igual que mira a las cabras del anciano.
```

Las demás piezas funcionan igual: un cofre ejecuta `action item.add`, activa una bandera y es sustituido por el evento del cofre abierto con la `Condition` contraria; el mercader ejecuta `action trade.open shop=rpg_demo/village_merchant`; la primera visita a la aldea lanza el escenario de introducción, que mete al héroe en el grupo con `action party.add id=rpg_hero`.

Para cambiar los mapas y eventos visualmente, usa el editor de mapas de las herramientas de edición del RPG.

## Si algo falla

1. **Una puerta no hace nada.** Una puerta `Touch` funciona al pisarla; una `Action` necesita E o F estando de frente.
2. **Un monstruo o un cofre volvió a aparecer.** Su `Condition` debe comprobar la misma bandera que activa el escenario.
3. **La tienda está vacía.** El mod trading debe estar en el juego, y el id de la tienda en el escenario debe coincidir con `shops/village_merchant.json`.
