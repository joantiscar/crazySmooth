<template>
    <div id="game"></div>
</template>
<script>
  /* eslint-disable */
  import Phaser from 'phaser'
  import tileset from '../assets/tileSets/tileset.png'
  import enemies_tileset from '../assets/tileSets/enemies.png'
  import map_json from '../assets/tileSets/map.json'
  import map2_json from '../assets/tileSets/map2.json'
  import jugador from '../assets/tileSets/player.png'
  import start from '../assets/start.png'
  import playMusic from '../assets/playMusic.png'
  import muteMusic from '../assets/muteMusic.png'
  import dust from '../assets/dust.png'
  import background from '../assets/background.jpg'
  import victoryBackground from '../assets/victoryBackground.png'
  import gameBackground from '../assets/gameBackground.png'
  import defeatBackground from '../assets/defeatBackground.jpg'
  import title from '../assets/title.png'
  import music from '../assets/music.mp3'
  import victoryMusic from '../assets/victoryMusic.mp3'
  import defeatMusic from '../assets/defeatMusic.mp3'
  import coinSound from '../assets/coin.mp3'
  import dustSound from '../assets/dust.wav'
  import jumpSound from '../assets/jump.mp3'
  import deadSound from '../assets/dead.mp3'
  import shine from '../assets/shine.png'
  import shapes_json from '../assets/particles/shapes.json'
  import shapes_png from '../assets/particles/shapes.png'


  var player;
  let coinLayer;
  let enemyLayer;
  let coins;
  let enemies;
  let backgroundMusic;
  let userWantsAudio;
  var spawnPoint;



  function createCoins() {
    coinLayer.forEach(object => {
      let obj = coins.create(object.x, object.y, 'tileset', 57)
      obj.body.width = object.width
      obj.body.height = object.height
    })
  }

  function takeCoin(player, coin) {
    coin.disableBody(true, true)
    player.score = player.score + 50
    this.sound.play('coinSound')
    var shine = this.add.particles('shine')
    this.shine = shine.createEmitter()
    this.shine.setPosition(coin.body.x, coin.body.y)
    this.shine.setSpeed(200)
    this.shine.setScale(0.1)
    this.shine.setBlendMode(Phaser.BlendModes.ADD)
    this.time.delayedCall(100, function () {
      shine.destroy()
    })
  }

  function moveEnemies() {
    for (let i = 1; i < 8; i++) {
      if (this['enemy' + i].body.blocked.right) {
        this['enemy' + i].flipX = true
      }
      if (this['enemy' + i].body.blocked.left) {
        this['enemy' + i].flipX = false
      }
      this['enemy' + i].body.velocity.x = 100 * (this['enemy' + i].flipX ? -1 : 1)
    }


  }


  function takeDamage(player, enemy) {
    var particles = this.add.particles('dust')
    this.dust = particles.createEmitter()
    this.dust.setPosition(player.body.x, player.body.y)
    this.dust.setSpeed(200)
    this.dust.setBlendMode(Phaser.BlendModes.ADD)
    this.time.delayedCall(400, function () {
      particles.destroy()
    })
    this.sound.play('deadSound')
    shake.call(this)
    respawn.call(this)
  }

  function respawn() {
    if (this.coins) {
      this.coins.clear(true, true)
    }
    this.enemy1.body.x = 300
    this.enemy2.body.x = 650
    this.enemy3.body.x = 500
    this.enemy4.body.x = 1100
    this.enemy5.body.x = 1500
    this.enemy6.body.x = 1700
    this.enemy7.body.x = 1000
    for (let i = 1; i < 8; i++) {
      this['enemy' + i].flipX = false
    }
    createCoins()
    spawnPoint = this.map.findObject('player', obj => obj.name === 'spanwPoint');

    player.body.x = spawnPoint.x
    player.body.y = spawnPoint.y
    player.lives--

    if (player.lives === 0) this.scene.start('Defeat')
    else {

    }


  }

  function shake() {
    this.cameras.main.shake(200)
  }

  function nextLevel(player, warp) {
    this.scene.start('Victory')
  }

  export default {
    name: 'Game',
    created() {

      let config = {
        type: Phaser.AUTO,
        scale: {
          mode: Phaser.Scale.FIT,
          parent: 'game',
          autoCenter: Phaser.Scale.CENTER_BOTH,
          width: window.innerWidth,
          height: window.innerHeight
        },
        scene: [Loading, Menu, Level1, Level2, Victory, Defeat],
        // scenes: [Loading, Menu, Level1, Level2, Victory, Defeat],
        physics: {
          default: 'arcade',
          arcade: {
            gravity: {
              y: 700
            }
          }
        },
        pixelArt: true
      }
      // eslint-disable-next-line
      new Phaser.Game(config)
    }
  }

  class Loading extends Phaser.Scene {
    constructor() {
      super({key: 'loading'})
    }

    preload() {
      console.log("PRELOAD");
      this.load.spritesheet('tileset', tileset, {frameWidth: 16, frameHeight: 16})
      this.load.spritesheet('enemies_tileset', enemies_tileset, {frameWidth: 16, frameHeight: 16})
      this.load.tilemapTiledJSON("map", map_json)
      this.load.tilemapTiledJSON("map2", map2_json)
      this.load.spritesheet('player', jugador, {frameWidth: 50, frameHeight: 37})
      this.load.image('start', start)
      this.load.image('muteMusic', muteMusic)
      this.load.image('playMusic', playMusic)
      this.load.image('background', background)
      this.load.image('victoryBackground', victoryBackground)
      this.load.image('shine', shine)
      this.load.image('defeatBackground', defeatBackground)
      this.load.image('gameBackground', gameBackground)
      this.load.image('title', title)
      this.load.image('dust', dust)
      this.load.audio('music', music)
      this.load.audio('victoryMusic', victoryMusic)
      this.load.audio('defeatMusic', defeatMusic)
      this.dustSound = this.load.audio('dustSound', dustSound)
      this.jumpSound = this.load.audio('jumpSound', jumpSound)
      this.coinSound = this.load.audio('coinSound', coinSound)
      this.deadSound = this.load.audio('deadSound', deadSound)
      this.load.atlas('neu', shapes_png, shapes_json)
      this.load.plugin('rexvirtualjoystickplugin', 'https://raw.githubusercontent.com/rexrainbow/phaser3-rex-notes/master/plugins/dist/rexvirtualjoystickplugin.min.js', true)
      var progressBar = this.add.graphics()
      var progressBox = this.add.graphics()
      progressBox.fillStyle(0x222222, 0.8)
      var width = this.cameras.main.width
      var height = this.cameras.main.height
      progressBox.fillRect(window.x = width * 0.5 - 165, window.y = height * 0.5 - 75, 320, 50)
      var loadingText = this.make.text({
        x: width * 0.5,
        y: height * 0.5 - 100,
        text: 'Loading...',
        style: {
          font: '20px monospace',
          fill: '#ffffff'
        }
      })
      loadingText.setOrigin(0.5, 0.5)
      var percentText = this.make.text({
        x: width * 0.5,
        y: height * 0.5 - 50,
        text: '0%',
        style: {
          font: '18px monospace',
          fill: '#ffffff'
        }
      })
      percentText.setOrigin(0.5, 0.5)
      var assetText = this.make.text({
        x: width * 0.5,
        y: height * 0.5 - 5,
        text: '',
        style: {
          font: '18px monospace',
          fill: '#ffffff'
        }
      })
      assetText.setOrigin(0.5, 0.5)
      this.load.on('progress', function (value) {
        percentText.setText(parseInt(value * 100) + '%')
        progressBar.clear()
        progressBar.fillStyle(0xffffff, 1)
        progressBar.fillRect(window.x = width * 0.5 - 165, window.y = height * 0.5 - 65, 300 * value, 30)
      })
      this.load.on('fileprogress', function (file) {
        assetText.setText('Loading asset: ' + file.key)
      })
      this.load.on('complete', function () {
        progressBar.destroy()
        progressBox.destroy()
        loadingText.destroy()
        percentText.destroy()
        assetText.destroy()
      })
    }

    create() {
      this.scene.start('menu')
    }
  }

  class Menu extends Phaser.Scene {
    constructor() {
      super({key: 'menu'})
    }

    create() {
      userWantsAudio = true
      backgroundMusic = this.sound.add('music')
      backgroundMusic.play()
      let background = this.add.image(0, 0, 'background').setOrigin(0).setDepth(0)
      background.displayHeight = this.game.renderer.height
      background.displayWidth = this.game.renderer.width
      let title = this.add.image(this.game.renderer.width / 2, this.game.renderer.height / 2 - (this.game.renderer.height / 3), 'title')

      title.displayHeight = this.game.renderer.height / 5
      title.displayWidth = this.game.renderer.width / 3

      let start = this.add.image(this.game.renderer.width / 2, this.game.renderer.height / 2, 'start').setDepth(1)
      start.displayHeight = 120
      start.displayWidth = 120
      start.setInteractive()
      start.on('pointerdown', () => {
        this.scene.start('Level1')
      })
      let playMusicButton = this.add.image(this.game.renderer.width / 2 - 100, this.game.renderer.height / 2, 'playMusic').setDepth(1)
      playMusicButton.displayHeight = 40
      playMusicButton.displayWidth = 40

      let muteButton = this.add.image(this.game.renderer.width / 2 + 100, this.game.renderer.height / 2, 'muteMusic').setDepth(1)
      muteButton.displayHeight = 40
      muteButton.displayWidth = 40

      playMusicButton.setInteractive()
      muteButton.setInteractive()

      muteButton.on('pointerdown', () => {
        backgroundMusic.play()
        backgroundMusic.pause()
        userWantsAudio = false
      })

      playMusicButton.on('pointerdown', () => {
        backgroundMusic.play()
        userWantsAudio = true
      })
    }
  }

  class Victory extends Phaser.Scene {
    constructor() {
      super({key: 'Victory'})
    }

    create() {
      let victoryScreenMusic = this.sound.add('victoryMusic')
      if (userWantsAudio) {
        backgroundMusic.stop()
        victoryScreenMusic.play()
      }

      let victoryBackground = this.add.image(0, 0, 'victoryBackground').setOrigin(0).setDepth(0)
      victoryBackground.displayHeight = this.game.renderer.height
      victoryBackground.displayWidth = this.game.renderer.width
      let start = this.add.image(this.game.renderer.width / 2, this.game.renderer.height / 2 + this.game.renderer.height / 4, 'start').setDepth(1)
      start.displayHeight = 120
      start.displayWidth = 120
      start.setInteractive()
      start.on('pointerdown', () => {
        if (userWantsAudio) {
          victoryScreenMusic.stop()
          backgroundMusic.play()

        }

        this.scene.start('Level1')
      })
    }
  }

  class Defeat extends Phaser.Scene {
    constructor() {
      super({key: 'Defeat'})
    }

    create() {
      let defeatScreenMusic = this.sound.add('defeatMusic')

      if (userWantsAudio) {
        backgroundMusic.stop()
        defeatScreenMusic.play()
      }

      let defeatBackground = this.add.image(0, 0, 'defeatBackground').setOrigin(0).setDepth(0)
      defeatBackground.displayHeight = this.game.renderer.height
      defeatBackground.displayWidth = this.game.renderer.width
      defeatBackground.setInteractive()
      defeatBackground.on('pointerdown', () => {
        if (userWantsAudio) {
          defeatScreenMusic.stop()
          backgroundMusic.play()
        }

        this.scene.start('Level1')
      })
    }
  }

  class Level1 extends Phaser.Scene {

    constructor() {
      super({key: 'Level1'})
    }

    preload() {
    }

    create() {
      const camera = this.cameras.main;
      if (this.sys.game.device.os.desktop) {
        this.cameras.main.setZoom(2)
      }
      let gameBackground = this.add.image(0, 0, 'gameBackground').setOrigin(0).setDepth(0)
      gameBackground.displayHeight = this.game.renderer.height
      gameBackground.displayWidth = this.game.renderer.width
      gameBackground.setScrollFactor(0)
      console.log("CREATED");
      this.map = this.make.tilemap({key: "map"})
      let tileset = this.map.addTilesetImage("tileset", "tileset")
      let enemies_tileset = this.map.addTilesetImage("enemies_tileset", "enemies_tileset")
      let floor = this.map.createStaticLayer("floor", tileset, 0, 0)
      let invisiblewalls = this.map.createStaticLayer("invisiblewalls", '', 0, 0)
      let elevation = this.map.createStaticLayer("elevation", tileset, 0, 0)
      let death = this.map.createStaticLayer("death", '', 0, 0)
      let warp = this.map.createStaticLayer("warp", tileset, 0, 0)
      let decoracio = this.map.createStaticLayer("decoracio", tileset, 0, 0)

      floor.setCollisionByExclusion([-1]);
      death.setCollisionByExclusion([-1]);
      warp.setCollisionByExclusion([-1]);
      elevation.setCollisionByExclusion([-1]);
      invisiblewalls.setCollisionByExclusion([-1]);
      coinLayer = this.map.getObjectLayer('items')['objects']

      var particlesNeu = this.add.particles('neu')
      particlesNeu.createEmitter(
        {
          "active": true,
          "visible": true,
          "collideBottom": true,
          "collideLeft": true,
          "collideRight": true,
          "collideTop": true,
          "on": true,
          "particleBringToTop": true,
          "radial": true,
          "frame": {"frames": ["star_05"], "cycle": false, "quantity": 1},
          "gravityY": 1200,
          "timeScale": 1,
          "alpha": 1,
          "angle": {"min": 0, "max": 360, "ease": "Linear"},
          "lifespan": 1000,
          "maxVelocityX": 10000,
          "maxVelocityY": 10000,
          "quantity": 1,
          "scale": 1,
          "x": 2,
          "y": 6,
          "tint": [16777215],
          "emitZone": {"source": new Phaser.Geom.Rectangle(0, 0, 3200, 50), "type": "random"}
      })


      // Habilitem un cursor per a les fletxes del teclat
      this.cursors = this.input.keyboard.createCursorKeys();

      this.cameras.main.backgroundColor.setTo(69, 911, 420)


      // La càmara no sortirà del món
      camera.setBounds(0, 0, this.map.widthInPixels, this.map.heightInPixels);

      this.spawnPoint = this.map.findObject('player', obj => obj.name === 'spanwPoint');


      player = this.physics.add.sprite(this.spawnPoint.x, this.spawnPoint.y, 'player');
      player.spawnPoint = this.spawnPoint
      player.lives = 3;
      //
      // // Animació de correr del player
      //
      this.physics.add.collider(player, floor)
      this.physics.add.collider(player, elevation)
      this.physics.add.collider(player, warp, nextLevel, null, this)
      this.physics.add.collider(player, death, takeDamage, null, this)
      elevation.setCollisionByExclusion([-1]);

      //
      player.body.setSize(24, 37, 12, 0)
      this.anims.create({
        key: 'idle',
        frames: this.anims.generateFrameNumbers('player', {start: 0, end: 3}),
        frameRate: 5,
        repeat: -1
      })
      this.anims.create({
        key: 'runRight',
        frames: this.anims.generateFrameNumbers('player', {start: 8, end: 13}),
        frameRate: 5,
        repeat: -1
      })
      this.anims.create({
        key: 'runLeft',
        frames: this.anims.generateFrameNumbers('player', {start: 8, end: 13}),
        frameRate: 5,
        repeat: -1
      })
      this.anims.create({
        key: 'jump',
        frames: this.anims.generateFrameNumbers('player', {start: 15, end: 23}),
        frameRate: 12,
        repeat: 0
      })
      camera.startFollow(player)
      player.isJumping = false
      coins = this.physics.add.staticGroup()
      enemies = this.physics.add.group()
      createCoins()
        player.score = 0

      this.enemy1 = this.physics.add.sprite(300, 460, 'enemies_tileset', 50)
      this.enemy2 = this.physics.add.sprite(650, 460, 'enemies_tileset', 50)
      this.enemy3 = this.physics.add.sprite(500, 460, 'enemies_tileset', 50)
      this.enemy4 = this.physics.add.sprite(1100, 460, 'enemies_tileset', 50)
      this.enemy5 = this.physics.add.sprite(1500, 460, 'enemies_tileset', 50)
      this.enemy6 = this.physics.add.sprite(1700, 460, 'enemies_tileset', 50)
      this.enemy7 = this.physics.add.sprite(1000, 460, 'enemies_tileset', 50)
      this.anims.create({
        key: 'enemyMove',
        frames: this.anims.generateFrameNumbers('enemies_tileset', {start: 50, end: 51}),
        frameRate: 6,
        repeat: -1
      })

      this.enemy1.body.velocity.x = 100
      this.enemy2.body.velocity.x = 143
      this.enemy3.body.velocity.x = 123
      this.enemy4.body.velocity.x = 67
      this.enemy5.body.velocity.x = 44
      this.enemy6.body.velocity.x = 234
      this.enemy7.body.velocity.x = 14
      for (let i = 1; i < 8; i++) {
        this.physics.add.collider(this['enemy' + i], floor)
        this.physics.add.collider(this['enemy' + i], elevation)
        this.physics.add.collider(this['enemy' + i], invisiblewalls)
        this.physics.add.collider(player, this['enemy' + i], takeDamage, null, this)
        this['enemy' + i].anims.play('enemyMove')
      }

      death.setCollisionByProperty({collides: true})

      this.physics.add.overlap(player, coins, takeCoin, null, this)
      this.physics.add.overlap(player, enemies, takeDamage, null, this)

      if (!this.sys.game.device.os.desktop) {
        this.joyStick = this.plugins.get('rexvirtualjoystickplugin').add(this, {
          x: this.cameras.main.centerX - 250,
          y: this.cameras.main.centerY + 100,
          radius: 40,
          base: this.add.graphics().fillStyle(0x888888).fillCircle(0, 0, 50),
          thumb: this.add.graphics().fillStyle(0xcccccc).fillCircle(0, 0, 30)
        })
        this.cursorKeysVirtual = this.joyStick.createCursorKeys()

      }
      if (!this.sys.game.device.os.desktop) {
        this.scoreText = this.add.text(100, 10, 'Puntuació: ' + player.score, {
          fontSize: '16px',
          fill: '#fff'
        }).setScrollFactor(0).setDepth(200)
        this.livesText = this.add.text(10, 10, 'Vides: ' + player.lives, {
          fontSize: '16px',
          fill: '#fff'
        }).setScrollFactor(0).setDepth(200)
      } else {
        this.scoreText = this.add.text((window.innerWidth / 3) - 100, (window.innerHeight / 4), 'Puntuació: ' + player.score, {
          fontSize: '16px',
          fill: '#fff'
        }).setScrollFactor(0).setDepth(200)
        this.livesText = this.add.text((window.innerWidth / 3) + 50, (window.innerHeight / 4), 'Vides: ' + player.lives, {
          fontSize: '16px',
          fill: '#fff'
        }).setScrollFactor(0).setDepth(200)
      }

    }

    update() {
      this.input.update()
      this.scoreText.setText('Puntuació: ' + player.score)
      this.livesText.setText('Vides: ' + player.lives)

      if (player.body.velocity.x === 0 && !player.isJumping) player.anims.play('idle', true)


      if (player.isJumping && player.isJumpingAnimation) {
        player.anims.play('jump', false)
        this.sound.play('jumpSound')

        player.isJumpingAnimation = false


      }

      // INPUT EVENTS

      if (this.sys.game.device.os.desktop) {

        if (this.cursors.left.isDown) {
          player.setVelocityX(-160)
          if (!player.isJumping) player.anims.play('runLeft', true)
          player.flipX = true


        } else if (this.cursors.right.isDown) {
          player.setVelocityX(160)
          if (!player.isJumping) player.anims.play('runRight', true)
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
      } else {
        if (this.cursorKeysVirtual.left.isDown) {
          player.setVelocityX(-160)
          if (!player.isJumping) player.anims.play('runLeft', true)
          player.flipX = true


        } else if (this.cursorKeysVirtual.right.isDown) {
          player.setVelocityX(160)
          if (!player.isJumping) player.anims.play('runRight', true)
          player.flipX = false


        } else {
          if (player.body.velocity.x > 0) player.setVelocityX(player.body.velocity.x - 20)
          if (player.body.velocity.x < 0) player.setVelocityX(player.body.velocity.x + 20)
          if (player.body.velocity.x < 20 && player.body.velocity.x > -20) player.setVelocityX(0)

        }
        if (this.cursorKeysVirtual.up.isDown && player.body.onFloor()) {
          player.setVelocityY(-340) // 340
          player.isJumping = true
          player.isJumpingAnimation = true

        }
      }

      if (player.body.onFloor() && player.body.velocity.y > -300) {
        if (player.isJumping) this.sound.play('dustSound')
        player.isJumping = false


      }

      moveEnemies.call(this)
    }
  }

  class Level2 extends Phaser.Scene {

    constructor() {
      super({key: 'Level2'})
    }
    create() {
      const camera = this.cameras.main;
      if (this.sys.game.device.os.desktop) {
        this.cameras.main.setZoom(2)
      }
      let gameBackground = this.add.image(0, 0, 'gameBackground').setOrigin(0).setDepth(0)
      gameBackground.displayHeight = this.game.renderer.height
      gameBackground.displayWidth = this.game.renderer.width
      gameBackground.setScrollFactor(0)
      this.map = this.make.tilemap({key: "map2"})
      let tileset = this.map.addTilesetImage("tileset", "tileset")
      let enemies_tileset = this.map.addTilesetImage("enemies_tileset", "enemies_tileset")
      let floor = this.map.createStaticLayer("floor", tileset, 0, 0)
      let invisiblewalls = this.map.createStaticLayer("invisiblewalls", '', 0, 0)
      let elevation = this.map.createStaticLayer("elevation", tileset, 0, 0)
      let death = this.map.createStaticLayer("death", '', 0, 0)
      let warp = this.map.createStaticLayer("warp", tileset, 0, 0)
      let decoracio = this.map.createStaticLayer("decoracio", tileset, 0, 0)

      floor.setCollisionByExclusion([-1]);
      death.setCollisionByExclusion([-1]);
      warp.setCollisionByExclusion([-1]);
      elevation.setCollisionByExclusion([-1]);
      invisiblewalls.setCollisionByExclusion([-1]);
      coinLayer = this.map.getObjectLayer('items')['objects']

      var particlesNeu = this.add.particles('neu')
      particlesNeu.createEmitter(
        {
          "active": true,
          "visible": true,
          "collideBottom": true,
          "collideLeft": true,
          "collideRight": true,
          "collideTop": true,
          "on": true,
          "particleBringToTop": true,
          "radial": true,
          "frame": {"frames": ["star_05"], "cycle": false, "quantity": 1},
          "gravityY": 1200,
          "timeScale": 1,
          "alpha": 1,
          "angle": {"min": 0, "max": 360, "ease": "Linear"},
          "lifespan": 1000,
          "maxVelocityX": 10000,
          "maxVelocityY": 10000,
          "quantity": 1,
          "scale": 1,
          "x": 2,
          "y": 6,
          "tint": [16777215],
          "emitZone": {"source": new Phaser.Geom.Rectangle(0, 0, 3200, 50), "type": "random"}
        })


      // Habilitem un cursor per a les fletxes del teclat
      this.cursors = this.input.keyboard.createCursorKeys();

      this.cameras.main.backgroundColor.setTo(69, 911, 420)


      // La càmara no sortirà del món
      camera.setBounds(0, 0, this.map.widthInPixels, this.map.heightInPixels);

      this.spawnPoint = this.map.findObject('player', obj => obj.name === 'spanwPoint');


      player = this.physics.add.sprite(this.spawnPoint.x, this.spawnPoint.y, 'player');
      player.spawnPoint = this.spawnPoint
      //
      // // Animació de correr del player
      //
      this.physics.add.collider(player, floor)
      this.physics.add.collider(player, elevation)
      this.physics.add.collider(player, warp, nextLevel, null, this)
      this.physics.add.collider(player, death, takeDamage, null, this)
      elevation.setCollisionByExclusion([-1]);

      //
      player.body.setSize(24, 37, 12, 0)
      this.anims.create({
        key: 'idle',
        frames: this.anims.generateFrameNumbers('player', {start: 0, end: 3}),
        frameRate: 5,
        repeat: -1
      })
      this.anims.create({
        key: 'runRight',
        frames: this.anims.generateFrameNumbers('player', {start: 8, end: 13}),
        frameRate: 5,
        repeat: -1
      })
      this.anims.create({
        key: 'runLeft',
        frames: this.anims.generateFrameNumbers('player', {start: 8, end: 13}),
        frameRate: 5,
        repeat: -1
      })
      this.anims.create({
        key: 'jump',
        frames: this.anims.generateFrameNumbers('player', {start: 15, end: 23}),
        frameRate: 12,
        repeat: 0
      })
      camera.startFollow(player)
      player.isJumping = false
      coins = this.physics.add.staticGroup()
      enemies = this.physics.add.group()
      createCoins()

      this.enemy1 = this.physics.add.sprite(580, 281, 'enemies_tileset',280)
      this.enemy2 = this.physics.add.sprite(770, 281, 'enemies_tileset',280)
      this.enemy3 = this.physics.add.sprite(957, 281, 'enemies_tileset',280)
      this.enemy4 = this.physics.add.sprite(1100, 281, 'enemies_tileset',280)
      this.enemy5 = this.physics.add.sprite(1500, 281, 'enemies_tileset',280)
      this.enemy6 = this.physics.add.sprite(1700, 281, 'enemies_tileset',280)
      this.enemy7 = this.physics.add.sprite(1000, 281, 'enemies_tileset',280)
      this.anims.create({
        key: 'enemyMove',
        frames: this.anims.generateFrameNumbers('enemies_tileset', {start: 280, end: 281}),
        frameRate: 6,
        repeat: -1
      })

      this.enemy1.body.velocity.x = 100
      this.enemy2.body.velocity.x = 143
      this.enemy3.body.velocity.x = 123
      this.enemy4.body.velocity.x = 67
      this.enemy5.body.velocity.x = 44
      this.enemy6.body.velocity.x = 234
      this.enemy7.body.velocity.x = 14
      for (let i = 1; i < 8; i++) {
        this.physics.add.collider(this['enemy' + i], floor)
        this.physics.add.collider(this['enemy' + i], elevation)
        this.physics.add.collider(this['enemy' + i], invisiblewalls)
        this.physics.add.collider(player, this['enemy' + i], takeDamage, null, this)
        this['enemy' + i].anims.play('enemyMove')
      }

      death.setCollisionByProperty({collides: true})

      this.physics.add.overlap(player, coins, takeCoin, null, this)
      this.physics.add.overlap(player, enemies, takeDamage, null, this)

      if (!this.sys.game.device.os.desktop) {
        this.joyStick = this.plugins.get('rexvirtualjoystickplugin').add(this, {
          x: this.cameras.main.centerX - 250,
          y: this.cameras.main.centerY + 100,
          radius: 40,
          base: this.add.graphics().fillStyle(0x888888).fillCircle(0, 0, 50),
          thumb: this.add.graphics().fillStyle(0xcccccc).fillCircle(0, 0, 30)
        })
        this.cursorKeysVirtual = this.joyStick.createCursorKeys()

      }
      if (!this.sys.game.device.os.desktop) {
        this.scoreText = this.add.text(100, 10, 'Puntuació: ' + player.score, {
          fontSize: '16px',
          fill: '#fff'
        }).setScrollFactor(0).setDepth(200)
        this.livesText = this.add.text(10, 10, 'Vides: ' + player.lives, {
          fontSize: '16px',
          fill: '#fff'
        }).setScrollFactor(0).setDepth(200)
      } else {
        this.scoreText = this.add.text((window.innerWidth / 3) - 100, (window.innerHeight / 4), 'Puntuació: ' + player.score, {
          fontSize: '16px',
          fill: '#fff'
        }).setScrollFactor(0).setDepth(200)
        this.livesText = this.add.text((window.innerWidth / 3) + 50, (window.innerHeight / 4), 'Vides: ' + player.lives, {
          fontSize: '16px',
          fill: '#fff'
        }).setScrollFactor(0).setDepth(200)
      }

    }

    update() {
      this.input.update()
      this.scoreText.setText('Puntuació: ' + player.score)
      this.livesText.setText('Vides: ' + player.lives)

      if (player.body.velocity.x === 0 && !player.isJumping) player.anims.play('idle', true)


      if (player.isJumping && player.isJumpingAnimation) {
        player.anims.play('jump', false)
        this.sound.play('jumpSound')

        player.isJumpingAnimation = false


      }

      // INPUT EVENTS

      if (this.sys.game.device.os.desktop) {

        if (this.cursors.left.isDown) {
          player.setVelocityX(-160)
          if (!player.isJumping) player.anims.play('runLeft', true)
          player.flipX = true


        } else if (this.cursors.right.isDown) {
          player.setVelocityX(160)
          if (!player.isJumping) player.anims.play('runRight', true)
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
      } else {
        if (this.cursorKeysVirtual.left.isDown) {
          player.setVelocityX(-160)
          if (!player.isJumping) player.anims.play('runLeft', true)
          player.flipX = true


        } else if (this.cursorKeysVirtual.right.isDown) {
          player.setVelocityX(160)
          if (!player.isJumping) player.anims.play('runRight', true)
          player.flipX = false


        } else {
          if (player.body.velocity.x > 0) player.setVelocityX(player.body.velocity.x - 20)
          if (player.body.velocity.x < 0) player.setVelocityX(player.body.velocity.x + 20)
          if (player.body.velocity.x < 20 && player.body.velocity.x > -20) player.setVelocityX(0)

        }
        if (this.cursorKeysVirtual.up.isDown && player.body.onFloor()) {
          player.setVelocityY(-340) // 340
          player.isJumping = true
          player.isJumpingAnimation = true

        }
      }

      if (player.body.onFloor() && player.body.velocity.y > -300) {
        if (player.isJumping) this.sound.play('dustSound')
        player.isJumping = false


      }

      moveEnemies.call(this)
    }
  }
</script>

