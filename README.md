# WEBS6 Werkcollege week 2 demo: Architecture
In deze demo gaan we de onderwerpen uit het werkcollege behandelen.
We gaan een applicatie maken met providers, verschillende modules en routing.

## Starter
In de starter vind je een applicatie die een lijst van 'superhelden' laat zien. 
Deze gaan we refactoren. 

## Providers
Op dit moment staat er in de het component 'Heroes' een lijst van helden. Deze gaan we verplaatsen naar een provider.

1. Open _/architecture/src/app/heroes/heroes.component.ts
2. Maak hier een nieuwe provider aan met de naam heroes.provider.ts

```javascript
import { Injectable } from '@angular/core';

@Injectable()
export class HeroProvider {

}
```
3. Maak in deze provider een lijst van helden en een methode om ze op te halen

```javascript
    private _heroes: Hero[];

    constructor(){
        this._heroes = [];
        this._heroes[0] = new Hero(1, "Bedmen");
        this._heroes[1] = new Hero(2, "Wonden woman");
        this._heroes[2] = new Hero(3, "Flesje");
        this._heroes[3] = new Hero(4, "Supermarktman");
        this._heroes[4] = new Hero(5, "Green Lantaarnpaal");
    }
    
    public getHeroes(){
        return this._heroes;
    }
```
4. Om de provider te kunnen gebruiken moet deze bekend zijn bij een module. Bijvoorbeeld de app.module

```javascript
  providers: [HeroProvider],
```
5. Als laatste stap kunnen we de provider injecteren en hergebruiken in het component.
```javascript
  constructor(private heroProvider: HeroProvider){
      this.heroes = this.heroProvider.getHeroes();
  }
```

### Modules
Je begint te merken dat we nu van alles aan het doen zijn met onze heroes. Het kan makkelijk zijn om een set van componenten en providers in een module te bundelen. 

1. Maak een nieuwe module met de naam HeroesModule
```javascript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';


@NgModule({
  declarations: [],
  imports: [
    BrowserModule
  ],
  providers: [],
})
export class HeroesModule { }
```
2. Plaats alle providers en componenten in deze module i.p.v de app.module
```javascript
  declarations: [HeroesComponent],
  imports: [
    BrowserModule
  ],
  providers: [HeroProvider],
```
3. Gebruik deze module in de app.module!
```javascript
@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
    BrowserModule,
    HeroesModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
```javascript
```
```javascript
```
