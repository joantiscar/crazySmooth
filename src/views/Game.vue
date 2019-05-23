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
          width: window.innerWidth-50,
          height: window.innerHeight-50,
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
              this.load.spritesheet('player',_PLAYER ,{ frameWidth: 50, frameHeight: 37 })

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

              this.cameras.main.backgroundColor.setTo(69, 911, 420)


              // La càmara no sortirà del món
              camera.setBounds(0, 0, map.widthInPixels, map.heightInPixels);

              let spawnpoint = map.findObject('player', obj => obj.name === 'spanwPoint');

              this.debugText = this.add.text(16, 16, 'isJumping: false', { fontSize: '32px', fill: '#000' });
              this.debugText.setColor('#FFFFFF')

              this.debugText.setScrollFactor(0)
              player = this.physics.add.sprite(spawnpoint.x, spawnpoint.y, 'player');
              player.lives = 3;
              //
              // // Animació de correr del player
              //
              this.physics.add.collider(player,floor)
              //
              player.body.setSize(24, 37, 12, 0)
              this.anims.create({
                key: 'idle',
                frames: this.anims.generateFrameNumbers('player', { start: 0, end: 3 }),
                frameRate: 5,
                repeat: -1
              })
              this.anims.create({
                key: 'runRight',
                frames: this.anims.generateFrameNumbers('player', { start: 8, end: 13 }),
                frameRate: 5,
                repeat: -1
              })
              this.anims.create({
                key: 'runLeft',
                frames: this.anims.generateFrameNumbers('player', { start: 8, end: 13 }),
                frameRate: 5,
                repeat: -1
              })
              this.anims.create({
                key: 'jump',
                frames: this.anims.generateFrameNumbers('player', { start: 15, end: 23 }),
                frameRate: 12,
                repeat: 0
              })
              camera.startFollow(player)
              player.isJumping = false

            },
            update () {
              if (player.body.velocity.x === 0 && !player.isJumping) player.anims.play('idle', true)

              this.debugText.setText('isJmping: ' + player.isJumping);

              if (player.isJumping && player.isJumpingAnimation) {
                player.anims.play('jump', false)
                player.isJumpingAnimation = false


              }

              // INPUT EVENTS
              if (this.cursors.left.isDown) {
                player.setVelocityX(-160)
                if (! player.isJumping) player.anims.play('runLeft', true)
                player.flipX = true


              } else if (this.cursors.right.isDown) {
                player.setVelocityX(160)
                if (! player.isJumping) player.anims.play('runRight', true)
                player.flipX = false



              } else {
                if (player.body.velocity.x > 0) player.setVelocityX(player.body.velocity.x - 20)
                if (player.body.velocity.x < 0) player.setVelocityX(player.body.velocity.x + 20)
                if (player.body.velocity.x < 20 && player.body.velocity.x > -20) player.setVelocityX(0)

              }
              if (this.cursors.up.isDown && player.body.onFloor()) {
                player.setVelocityY(-340) // 340
                player.isJumping = true
                player.isJumpingAnimation = true

              }

              if (player.body.onFloor() && player.body.velocity.y > -300) {
                player.isJumping = false
              }
            }
          }
        }
        // eslint-disable-next-line
        new Phaser.Game(config)
      }
    }
  </script>

