<template>
  <div id="game"></div>
</template>
<script>
  import Phaser from 'phaser'
  import tileSet from './assets/tileset.png'
  import itemSet from './assets/tileset.png'
  import firstmap_Tiles from './assets/maps/firstmap_Tiles.csv'
  import firstmap_Items from './assets/maps/firstmap_Items.csv'

  export default {
    name: 'Game',
    mounted () {
      // PHASER 2.6 -> phaser-ce
      // Phaser 3.0 -> phaser

      let config = {
        type: Phaser.AUTO,
        width: 800,
        height: 600,
        physics: {
          default: 'arcade',
          arcade: {
            gravity: {
              y: 100
            }
          }
        },
        // NO HI HA STATES A 3.0 -> SCENES
        scene: {
          preload() {
            console.log('PRELOAD')
            this.load.image('tileset', tileset);
            this.load.tilemapCSV('firstmap_Items', firstmap_Items)
            this.load.tilemapCSV('firstmap_Tiles', firstmap_Tiles)

          },
          create() {
            let map = this.make.tilemap({key: "map", tileWidth: 16, tileHeight: 16});
            let tileSet = map.addTilesetImage("tiles");
            let itemSet = map.addTilesetImage("items");
            let tilesLayer = map.createStaticLayer(0, tileSet, 0, 0); // layer index, tileset, x, y
            let itemsLayer = map.createStaticLayer(0, itemSet, 0, 0); // layer index, tileset, x, y
            console.log('CREATE')
            // INITIALIZE del nivell -> Afegirem tiles (level: pareds, terres), afegirem players, collectibles (coins)
            this.cameras.main.backgroundColor.setTo(69, 911, 420)

            this.cursors = this.input.keyboard.createCursorKeys()

          },
          update() {
          }
        }
      }
      // eslint-disable-next-line
      new Phaser.Game(config)
    },
    methods: {}
  }
</script>
<style scoped></style>
