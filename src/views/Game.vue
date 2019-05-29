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
  import start from '../assets/start.png'
  import playMusic from '../assets/playMusic.png'
  import muteMusic from '../assets/muteMusic.png'
  import background from '../assets/background.jpg'
  import victoryBackground from '../assets/victoryBackground.png'
  import defeatBackground from '../assets/defeatBackground.jpg'
  import title from '../assets/title.png'
  import music from '../assets/music.mp3'
  import victoryMusic from '../assets/victoryMusic.mp3'
  import defeatMusic from '../assets/defeatMusic.mp3'

  var player;
  let coinLayer;
  let enemyLayer;
  let coins;
  let enemies;
  let backgroundMusic;
  let userWantsAudio;

  function createCoins() {
    coinLayer.forEach(object => {
      let obj = coins.create(object.x, object.y, 'tileset', 57)
      obj.body.width = object.width
      obj.body.height = object.height
    })
  }

  function takeCoin(player, coin) {
    coin.disableBody(true, true)

  }

  function createEnemies() {
    enemyLayer.forEach(object => {
      let tile
      console.log(object)
      if (object.properties[0].value === "goomba") tile = 50
      let obj = enemies.create(object.x, object.y, 'enemies_tileset', tile)
      obj.body.width = object.width
      obj.body.height = object.height
    })
  }

  function takeDamage(player, enemy) {

    console.log('Level2')
  }

  function fall(player, fall) {

    this.scene.start('Defeat')
  }

  function nextLevel(player, warp) {
    this.scene.start('Victory')
  }

  export default {
    name: 'Game',
    created() {

      let config = {
        type: Phaser.AUTO,
        width: window.innerWidth - 50,
        height: window.innerHeight - 50,
        scene: [Loading, Menu, Level1, Level2, Victory, Defeat],
        // scenes: [Loading, Menu, Level1, Level2, Victory, Defeat],
        physics: {
          default: 'arcade',
          arcade: {
            gravity: {
              y: 700
            },
            debug: true
          }
        }
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
      this.load.spritesheet('player', jugador, {frameWidth: 50, frameHeight: 37})
      this.load.image('start', start)
      this.load.image('muteMusic', muteMusic)
      this.load.image('playMusic', playMusic)
      this.load.image('background', background)
      this.load.image('victoryBackground', victoryBackground)
      this.load.image('defeatBackground', defeatBackground)
      this.load.image('title', title)
      this.load.audio('music', music)
      this.load.audio('victoryMusic', victoryMusic)
      this.load.audio('defeatMusic', defeatMusic)


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

      this.add.image(this.game.renderer.width / 2, this.game.renderer.height / 2 - 200, 'title')


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
        if (userWantsAudio){
          backgroundMusic.stop()
          victoryScreenMusic.play()
        }

      let victoryBackground = this.add.image(0, 0, 'victoryBackground').setOrigin(0).setDepth(0)

      let start = this.add.image(this.game.renderer.width / 2, this.game.renderer.height / 2 + 200, 'start').setDepth(1)
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

      if (userWantsAudio){
        backgroundMusic.stop()
        defeatScreenMusic.play()
      }

      let defeatBackground = this.add.image(0, 0, 'defeatBackground').setOrigin(0).setDepth(0)

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
      console.log("CREATED");
      let map = this.make.tilemap({key: "map"})
      let tileset = map.addTilesetImage("tileset", "tileset")
      let enemies_tileset = map.addTilesetImage("enemies_tileset", "enemies_tileset")
      let floor = map.createStaticLayer("floor", tileset, 0, 0)
      let elevation = map.createStaticLayer("elevation", tileset, 0, 0)
      let death = map.createStaticLayer("death", '', 0, 0)
      let warp = map.createStaticLayer("warp", tileset, 0, 0)

      floor.setCollisionByExclusion([-1]);
      death.setCollisionByExclusion([-1]);
      warp.setCollisionByExclusion([-1]);
      elevation.setCollisionByExclusion([-1]);
      coinLayer = map.getObjectLayer('items')['objects']
      enemyLayer = map.getObjectLayer('enemies')['objects']


      // Habilitem un cursor per a les fletxes del teclat
      this.cursors = this.input.keyboard.createCursorKeys();

      this.cameras.main.backgroundColor.setTo(69, 911, 420)


      // La càmara no sortirà del món
      camera.setBounds(0, 0, map.widthInPixels, map.heightInPixels);

      let spawnpoint = map.findObject('player', obj => obj.name === 'spanwPoint');

      this.debugText = this.add.text(16, 16, 'isJumping: false', {fontSize: '32px', fill: '#000'});
      this.debugText.setColor('#FFFFFF')

      this.debugText.setScrollFactor(0)
      player = this.physics.add.sprite(spawnpoint.x, spawnpoint.y, 'player');
      player.lives = 3;
      //
      // // Animació de correr del player
      //
      this.physics.add.collider(player, floor, function () {
        console.log('ESTIC AN TERRA')

      })
      this.physics.add.collider(player, elevation)
      this.physics.add.collider(player, warp, nextLevel, null, this)
      this.physics.add.collider(player, death, fall, null, this)
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
      enemies = this.physics.add.staticGroup()
      createCoins()
      createEnemies()
      death.setCollisionByProperty({collides: true})

      this.physics.add.overlap(player, coins, takeCoin, null, this)
      this.physics.add.overlap(player, enemies, takeDamage, null, this)

    }

    update() {
      if (player.body.velocity.x === 0 && !player.isJumping) player.anims.play('idle', true)

      this.debugText.setText('isJmping: ' + player.isJumping);

      if (player.isJumping && player.isJumpingAnimation) {
        player.anims.play('jump', false)
        player.isJumpingAnimation = false


      }

      // INPUT EVENTS
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

      if (player.body.onFloor() && player.body.velocity.y > -300) {
        player.isJumping = false
      }
    }
  }

  class Level2 extends Phaser.Scene {

    constructor() {
      super({key: 'Level2'})
    }

    preload() {
      console.log("PRELOAD");
      this.load.spritesheet('tileset', tileset, {frameWidth: 16, frameHeight: 16})
      this.load.spritesheet('enemies_tileset', enemies_tileset, {frameWidth: 16, frameHeight: 16})
      this.load.tilemapTiledJSON("map", map_json)
      this.load.spritesheet('player', jugador, {frameWidth: 50, frameHeight: 37})

    }

    create() {
      const camera = this.cameras.main;
      console.log("CREATED");
      let map = this.make.tilemap({key: "map"})
      let tileset = map.addTilesetImage("tileset", "tileset")
      let enemies_tileset = map.addTilesetImage("enemies_tileset", "enemies_tileset")
      let floor = map.createStaticLayer("floor", tileset, 0, 0)
      let elevation = map.createStaticLayer("elevation", tileset, 0, 0)
      let death = map.createStaticLayer("death", '', 0, 0)
      let warp = map.createStaticLayer("warp", tileset, 0, 0)

      floor.setCollisionByExclusion([-1]);
      death.setCollisionByExclusion([-1]);
      warp.setCollisionByExclusion([-1]);
      elevation.setCollisionByExclusion([-1]);
      coinLayer = map.getObjectLayer('items')['objects']
      enemyLayer = map.getObjectLayer('enemies')['objects']


      // Habilitem un cursor per a les fletxes del teclat
      this.cursors = this.input.keyboard.createCursorKeys();

      this.cameras.main.backgroundColor.setTo(69, 911, 420)


      // La càmara no sortirà del món
      camera.setBounds(0, 0, map.widthInPixels, map.heightInPixels);

      let spawnpoint = map.findObject('player', obj => obj.name === 'spanwPoint');

      this.debugText = this.add.text(16, 16, 'isJumping: false', {fontSize: '32px', fill: '#000'});
      this.debugText.setColor('#FFFFFF')

      this.debugText.setScrollFactor(0)
      player = this.physics.add.sprite(spawnpoint.x, spawnpoint.y, 'player');
      player.lives = 3;
      //
      // // Animació de correr del player
      //
      this.physics.add.collider(player, floor, function () {
        console.log('ESTIC AN TERRA')

      })
      this.physics.add.collider(player, elevation)
      this.physics.add.collider(player, warp, nextLevel, null, this)
      this.physics.add.collider(player, death, fall, null, this)
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
      enemies = this.physics.add.staticGroup()
      createCoins()
      createEnemies()
      death.setCollisionByProperty({collides: true})

      this.physics.add.overlap(player, coins, takeCoin, null, this)
      this.physics.add.overlap(player, enemies, takeDamage, null, this)

    }

    update() {
      if (player.body.velocity.x === 0 && !player.isJumping) player.anims.play('idle', true)

      this.debugText.setText('isJmping: ' + player.isJumping);

      if (player.isJumping && player.isJumpingAnimation) {
        player.anims.play('jump', false)
        player.isJumpingAnimation = false


      }

      // INPUT EVENTS
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

      if (player.body.onFloor() && player.body.velocity.y > -300) {
        player.isJumping = false
      }
    }
  }
</script>

