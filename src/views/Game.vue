<template>
  <div id="game"></div>
</template>
  <script>
    /* eslint-disable */
    import Phaser from 'phaser'
    import tileset from '../assets/tileSets/tileset.png'
    import enemies_tileset from '../assets/tileSets/enemies.png'
    import map_json from '../assets/tileSets/map.json'
    import jugador from '../assets/tileSets/player.png'
    var player;
    let coinLayer;
    let enemyLayer;
    let coins;
    let enemies;

    function createCoins () {
      coinLayer.forEach(object => {
        let obj = coins.create(object.x, object.y, 'tileset',57)
        obj.body.width = object.width
        obj.body.height = object.height
      })
    }
    function  takeCoin(player, coin) {
      coin.disableBody(true,true)

    }
    function createEnemies () {
      enemyLayer.forEach(object => {
        let tile
        console.log(object)
        if (object.properties[0].value === "goomba") tile = 50
        let obj = enemies.create(object.x, object.y, 'enemies_tileset',tile)
        obj.body.width = object.width
        obj.body.height = object.height
      })
    }
    function  takeDamage(player, enemy) {
      enemy.disableBody(true,true)

    }
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
              this.load.spritesheet('tileset', tileset, {frameWidth: 16, frameHeight: 16})
              this.load.spritesheet('enemies_tileset', enemies_tileset, {frameWidth: 16, frameHeight: 16})
              this.load.tilemapTiledJSON("map",map_json)
              this.load.spritesheet('player',jugador ,{ frameWidth: 50, frameHeight: 37 })

            },
            create() {
              const camera = this.cameras.main;
              console.log("CREATED");
              let map = this.make.tilemap({ key: "map" })
              let tileset = map.addTilesetImage("tileset", "tileset")
              let enemies_tileset = map.addTilesetImage("enemies_tileset", "enemies_tileset")
              let floor = map.createStaticLayer("floor", tileset, 0, 0)
              let elevation = map.createStaticLayer("elevation",tileset,0,0)

              floor.setCollisionByExclusion([-1]);
              elevation.setCollisionByExclusion([-1]);
              coinLayer = map.getObjectLayer('items')['objects']
              enemyLayer = map.getObjectLayer('enemies')['objects']


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
              this.physics.add.collider(player,elevation)

              elevation.setCollisionByExclusion([-1]);

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
              coins = this.physics.add.staticGroup()
              enemies = this.physics.add.staticGroup()
              createCoins()
              createEnemies()
              this.physics.add.overlap(player,coins,takeCoin,null,this)
              this.physics.add.overlap(player,enemies,takeDamage,null,this)

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

