# Physics met Matter.js, Typescript en Parcel

[MatterJS](https://brm.io/matter-js/) berekent physics simulaties. [Bekijk de demo's](https://brm.io/matter-js/demo/#avalanche) en de [broncode van de demo's](https://github.com/liabru/matter-js/blob/master/examples/avalanche.js)

[ParcelJS](https://parceljs.org) bouwt een project van je `dev` map. Daarbij worden modules van MatterJS en je eigen project samengevoegd. Parcel zet Typescript om naar Javascript. Ook je afbeeldingen, css en html worden overgezet naar de `dist` folder!

<br>
<br>
<br>

## Installeren 

Installeer de Parcel module bundler globaal met `-g`. Dit zorgt dat je MatterJS modules kan gebruiken! Op de mac kan je er `sudo` voor zetten, en op windows kan je `run as administrator` doen, als je iets globaal wil installeren.

```bash
npm install -g parcel-bundler
```

Installeer de benodigde modules met

```bash
npm install
```

<br>
<br>
<br>

## Project openen

Tijdens development bekijk je je voortgang met

```bash
npm run dev
```
Als je project af is kan je de `dist` map bouwen met
```bash
npm run build
```



<br>
<br>
<br>

# Physics

Met Matter.JS maak je een *physics world*. Hierin wordt de positie van elementen uitgerekend door de physics engine. Elementen krijgen **gravity, collisions, force, velocity en friction**.

De physics world speelt zich puur af in het geheugen, er wordt niks getekend als je geen renderer toevoegt. In dit voorbeeld tekenen we DOM elementen op de positie van de elementen in de physics world.

Je kan de physics world 60 keer per seconde updaten, voordat je de posities gaat uitlezen:

```typescript
private engine : Matter.Engine
 
private setupMatter(){
    this.engine = Matter.Engine.create()
}

private gameLoop(){
    Matter.Engine.update(this.engine, 1000 / 60) 
    requestAnimationFrame(() => this.gameLoop())
}
```
<br>
<br>
<br>

## Physics objecten toevoegen

Je kan [default shapes](https://brm.io/matter-js/docs/classes/Bodies.html) zoals **rectangles en circles** toevoegen aan de physics world. 

Een rectangle verwacht een `x, y, width, height`. Een circle verwacht een `x,y,radius`. Let op dat `x,y` het middelpunt van het object is.

Een `static` shape ondervindt geen forces maar geeft wel collisions aan andere objecten. Dit gebruik je voor platforms en de vloer.

```typescript
let boxOne = Matter.Bodies.rectangle(0, 0, 50, 50)
let boxTwo = Matter.Bodies.rectangle(0, 500, 500, 10, {isStatic:true})
Matter.Composite.add(this.engine.world, [boxOne, boxTwo])
```

<br>
<br>
<br>

## DOM

We gaan de physics posities gebruiken om DOM elementen te positioneren. Daarvoor moeten we wel onthouden welk DOM element bij welk physics element hoort. Dat doen we in de class die ook het DOM element maakt:

```typescript
class Box {
    constructor(){
        // create physics object
        this.physicsBox = ...

        // create div
        this.div = ...
    }
}
```
De `Box` class kan nu de positie van het physics object opvragen, en daar het DOM element neer zetten. 

Let hierbij op dat een physics element zijn *center position* terug geeft. Om die reden moet je de `x,y` nog aanpassen met de hoogte en breedte van het object:

```typescript
let pos = this.physicsBox.position       // center point
let angle = this.physicsBox.angle        // hoek
let degrees = angle * (180 / Math.PI)

// div positioneren
this.div.style.transform = `translate(${pos.x - (this.width/2)}px, ${pos.y-(this.height/2)}px) rotate(${degrees}deg)`
```

<br>
<br>
<br>

## Beweging

Om te bewegen gebruik je 

- `Matter.body.translate` - Hiermee "hardcode" je de nieuwe positie ongeacht physics. Dit gebruik je voor **static** elementen zoals een platform.
- `Matter.Body.setVelocity` - Hiermee zet je een nieuwe snelheid. Bv. om een **player** of een **enemy** naar links / rechts te bewegen.
- `Matter.Body.addForce` - Hiermee geef je een object een eenmalige "boost", bv. een **raket** of **kogel** die je afvuurt, of een player die **springt**.

<br>

### Voorbeeld lopen

```typescript
Matter.Body.setVelocity(this.physicsBox, { x: this.speed, y: this.physicsBox.velocity.y })
```

### Voorbeeld springen

```typescript
Matter.Body.applyForce(this.physicsBox, { x: this.physicsBox.position.x, y: this.physicsBox.position.y }, { x: 0, y: -0.15 })
```

- [In deze tutorial vind je voorbeelden van alle drie methodes](https://code.tutsplus.com/tutorials/getting-started-with-matterjs-body-module--cms-28835)
- [Keyboard listeners voorbeeld](https://github.com/HR-CMGT/Typescript/blob/master/snippets/movement.md)

<br>
<br>
<br>

## Friction, gravity, bouncyness

Je kan de `gravity` van de wereld op x, y, of geen zetten, afhankelijk van het type spel. Een space game of een top-down view heeft geen gravity.

```typescript
this.engine = Matter.Engine.create()
this.engine.world.gravity.x = 1
this.engine.world.gravity.y = 0
this.engine.world.gravity.scale = 0.001
```

Je kan elementen (en zelfs de lucht) `friction` geven. Je kan elementen `restitution` geven om ze te laten stuiteren.

<br>
<br>
<br>

## Collision

Geef eerst een `label` mee in het options object om te weten welk physics element een collision heeft.

```typescript
let player = Matter.Bodies.circle(0, 0, 14, { label: 'player' })
```
Vervolgens kan je via het `onCollisionStart` event kijken welke elementen met elkaar botsen:
```typescript
Matter.Events.on(engine, 'collisionStart', (event) => {
    let collision = event.pairs[0]
    let bodyA = collision.bodyA
    let bodyB = collision.bodyB

    if (bodyA.label === 'player') {
        switch (bodyB.label) {
        case 'enemy':
            score--
            break;
        case 'food':
            score++
            break;
        }
    }
})
```
<br>
<br>
<br>

## Canvas

MatterJS heeft een basic "Render" functie. Die gebruiken we nu niet omdat we onze eigen DOM elementen gebruiken. [Je kan ook de matterjs physics rechtstreeks tekenen in een canvas](https://codepen.io/eerk/pen/xxqRpBL).

<br>
<br>
<br>

## Links

- [Demo's](https://brm.io/matter-js/demo/) met [broncode](https://github.com/liabru/matter-js/blob/master/examples/avalanche.js)
- [Documentatie](http://brm.io/matter-js/docs/)
- [Rectangles, Circles, physics Bodies](https://brm.io/matter-js/docs/classes/Bodies.html)
- [Codepen voorbeelden force en velocity](https://code.tutsplus.com/tutorials/getting-started-with-matterjs-body-module--cms-28835)
- [Tutorial](https://codersblock.com/blog/javascript-physics-with-matter-js/) met [Codepen Pinball voorbeeld](https://codepen.io/lonekorean/pen/KXLrVX)
- [Typescript definitions](https://www.npmjs.com/package/@types/matter-js)
- [Collision plugin - geef collision event listener aan een specifiek element](https://github.com/dxu/matter-collision-events)

<br>
<br>
<br>

## Bugs

ParcelJS CSSNano bug. Tijdelijk gefixed met `--no-minify`.
Todo: downgrade parcel naar `1.8.1`.