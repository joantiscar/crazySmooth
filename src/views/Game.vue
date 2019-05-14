<template>
  <div id="game"></div>
</template>
  <script>
    /* eslint-disable */
    import Phaser from 'phaser'
    import _TILESET_SET from '../assets/tileSets/tileset.png'
    import _JSON_MAP from '../assets/tileSets/map.json'
    import _PLAYER from '../assets/tileSets/player.png'
    var player;


    export default {
      name: 'Game',
      created() {
        var controls = false
        // PHASER 2.6 -> phaser-ce
        // PHASER 3.0 -> phaser
        let config = {
          type: Phaser.AUTO,
          width: window.innerWidth-25,
          height: window.innerHeight-20,
          physics: {
            default: 'arcade',
            arcade: {
              gravity: {
                y:  700
              },
              debug :true
            }
          },
          // NO HI HA STATES A 3.0 -> SCENES
          scene: {
            preload() {
              console.log("PRELOAD");
              this.load.image('tileset', _TILESET_SET)
              this.load.tilemapTiledJSON("map",_JSON_MAP)
              this.load.spritesheet('player',_PLAYER ,{ frameWidth: 28, frameHeight: 22 })

            },
            create() {
              const camera = this.cameras.main;
              console.log("CREATED");
              let map = this.make.tilemap({ key: "map" })
              let tileset = map.addTilesetImage("tileset", "tileset")
              let floor = map.createStaticLayer("floor", tileset, 0, 0)
              let elevation = map.createStaticLayer("elevation",tileset,0,0)

              floor.setCollisionByExclusion([-1]);
              elevation.setCollisionByExclusion([-1]);


              // Habilitem un cursor per a les fletxes del teclat
              this.cursors = this.input.keyboard.createCursorKeys();

              // La càmara no sortirà del món
              camera.setBounds(0, 0, map.widthInPixels, map.heightInPixels);

              let spawnpoint = map.findObject('player', obj => obj.name === 'spanwPoint');

              player = this.physics.add.sprite(spawnpoint.x, spawnpoint.y, 'player');
              player.setScale(2);
              player.lives = 3;
              //
              // // Animació de correr del player
              //
              this.physics.add.collider(player,floor)
              //
              this.anims.create({
                key: 'idle',
                frames: this.anims.generateFrameNumbers('player', { start: 3, end: 5 }),
                frameRate: 5,
                repeat: -1
              })
              camera.startFollow(player)

            },
            update () {
              player.anims.play('idle', true)

              // ESTE EL QUE S'executa continuament al Game loop -> 60 vegades per segon o FPS

              // INPUT EVENTS
              if (this.cursors.left.isDown) {
                player.setVelocityX(-160)
                player.setFrame(2)
              } else if (this.cursors.right.isDown) {
                player.setVelocityX(160)
                player.setFrame(1)
              } else {
                if (player.body.velocity.x > 0) player.setVelocityX(player.body.velocity.x - 1)
                if (player.body.velocity.x < 0) player.setVelocityX(player.body.velocity.x + 1)

                player.setFrame(0)
              }
              if (this.cursors.up.isDown && player.body.onFloor()) {
                player.setVelocityY(-200)
              }

              if (player.body.touching.down && player.y > 10) {
                // this.dustSound.play()
              }
            }
          }
        }
        // eslint-disable-next-line
        new Phaser.Game(config)
      }
    }
  </script>

