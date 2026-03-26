<a id="top"></a>

# Angular

(Stand März 2026) - Version 21

# Inhaltsverzeichnis

- [Grundlagen & Setup](#grundlagen--setup)
    - [Quickstart](#quickstart)
    - [Automatisiertes Bauen](#automatisiertes-bauen)
    - [Namenskonvention](#namenskonvention)
- [Architektur & Bootstrapping](#architektur--bootstrapping)
- [Wichtige Konzepte](#wichtige-konzepte)
    - [Components](#components)
    - [Service](#service)
    - [RxJS](#rxjs)
    - [Signals](#signals)
    - [Routing](#routing)
    - [HTTP Requests](#http-requests)
    - [Lazy Loading](#lazy-loading)
    - [Forms](#forms)
    - [Testing](#testing)
    - [Guards](#guards)
    - [Pipes](#pipes)
    - [Interceptor](#interceptor)
    - [Lifecycle Hooks](#lifecycle-hooks)
    - [Bindings / Directive](#bindings--directive)
    - [Beziehungen zwischen Components](#beziehungen-zwischen-components)
    - [DTO / Model](#dto--model)
- [Angular CLI (ab Version 21)](#angular-cli-ab-version-21)

# Grundlagen & Setup

https://angular.dev/overview   
https://www.angulararchitects.io/blog/angular-tutorial-teil-1-werkzeuge-und-erste-schritte/

Voraussetzung: Node JS, Angular CLI

**ACHTUNG:**  Der gradle build ist so konfiguriert, dass er die Angular App automatisch baut.   
Es ist also nicht notwendig, Node, Angular CLI manuell zu installieren oder zu verwenden.

**Node JS: (Global installieren)**   
Node JS installieren: https://nodejs.org/en/   
Version prüfen: `node -v` in Konsole

**Oder NVM: Node JS verwalten**   
Node Versionen verwalten: `nvm` (Node Version Manager) installieren
- `nvm use 18.17.1` oder `nvm use 18` (Wechseln zu Node Version 18.17.1)
- `nvm install 18.17.1` (Installieren von Node Version 18.17.1)
- `nvm list` (Liste aller installierten Node Versionen)
- `nvm -v` (Version von nvm prüfen)

**Angular CLI:**  
Tool um Angular Projekte zu bauen   
Installieren: `npm install -g @angular/cli`   
Version prüfen: `ng version`

<p align="right">(<a href="#top">nach oben</a>)</p>

---
## Quickstart

Neues Projekt anlegen:   
Öffne cmd und navigiere in den Ordner, wo der Workspace liegen soll.   
`ng new myProject`

**Verzeichnisstruktur:**
- node_modules: alle zusätzliche Libraries, welches das Angular Projekt verwendet (nicht manuell bearbeiten)
- src/app: Quellcode der Anwendung
- src/assets: Bilder, Videos, Sounds etc.
- environment-Dateien: Alle Infos zur App, die sich ändern z.B. Server URL, API Schlüssel
- index.html: Grundgerüst, welches die app-module einbindet
- main.ts und polyfills.ts: braucht man normalerweise nicht anpassen
- styles.css: CSS Code, der für alle Components gelten soll
- app.modules.ts: Tiefste Ebene wo alle Components, Services etc. für die Anwendung definiert sind. Dort kann man z.B. auch MaterialDesign importieren

Serve the application:
```
cd myProject   
ng serve --open   
```

`ng serve` macht folgendes:
1) App bauen
2) Development Server starten
3) Betrachtet die Sourcen
4) Baut die Anwendung erneut, wenn etwas geändert wurde

`--open` öffnet den Standardbrowser zu http://localhost:4200

Die Seite, die man jetzt in http://localhost:4200 sieht ist die sogenannte "application Shell".   
Die Shell wird von einer Angular Component kontrolliert. Die Component heißt "AppComponent".

Öffne das Verzeichnis in einem Code Editor   
Die AppComponent findet sich in `src/app` (app.component.ts, *.html, *.css)   
Den Titel kann man in der `.ts` ändern und den Inhalt der html löschen und `<h1>{{title}}</h1>`   
Seite wird automatisch aktualisiert.


<p align="right">(<a href="#top">nach oben</a>)</p>

___
## Automatisiertes Bauen

Aufbau des Projekts:
```
<root>
 ├── openapi/
 │    └── openapi.yaml
 ├── frontend/
 │    ├── package.json
 │    ├── angular.json
 │    ├── tsconfig.json
 │    └── src/
 │        └── app/
 │            └── openapi/ // automatisch generierte API-Client Dateien
 ├── build.gradle.kts
 ├── build/
 ```

1. Voraussetzung `package.json` im Projektverzeichnis
```json
{
  "name": "my-project",
  "version": "0.0.1",
  "scripts": {
    "ng": "ng",
    "start": "ng serve --open",
    "build": "ng build",
    "watch": "ng build --watch --configuration development",
    "test": "ng test",
    "lint": "ng lint",
    "e2e": "ng e2e"
  },
  "private": true,
  "dependencies": {
    "@angular/animations": "^21.2.0",
    "@angular/common": "^21.2.0",
    "@angular/compiler": "^21.2.0",
    "@angular/core": "^21.2.0",
    "@angular/forms": "^21.2.0",
    "@angular/platform-browser": "^21.2.0",
    "@angular/platform-browser-dynamic": "^21.2.0",
    "@angular/router": "^21.2.0",
    "rxjs": "~7.8.0",
    "tslib": "^2.3.0",
    "zone.js": "~0.16.0"
  }
}
```

2. `build.gradle.kts`
```groovy
import com.github.gradle.node.npm.task.NpmTask
import org.openapitools.generator.gradle.plugin.tasks.GenerateTask

plugins {
    id("org.openapi.generator") version "7.3.0"
    id("com.github.node-gradle.node") version "7.0.2"
}

val openApiSpec = "$rootDir/openapi/openapi.yaml"

//  Backend API bauen (Spring Boot Interfaces)
val generateBackend = tasks.register<GenerateTask>("generateBackend") {
    generatorName.set("kotlin-spring")
    inputSpec.set(openApiSpec)
    outputDir.set(layout.buildDirectory.dir("generated/backend").get().asFile.absolutePath)

    apiPackage.set("com.example.api")
    modelPackage.set("com.example.model")

    configOptions.set(
            mapOf(
                    "delegatePattern" to "true",
                    "interfaceOnly" to "false",
                    "useSpringBoot3" to "true",
                    "useTags" to "true",
                    "generateApiDocumentation" to "true",
                    "generateModelDocumentation" to "true",
                    "serializationLibrary" to "kotlinx_serialization",
                    "dateLibrary" to "java8",
                    "hideGenerationTimestamp" to "true",
            )
    )
}

//  Frontend API Client
val generateFrontend = tasks.register<GenerateTask>("generateFrontend") {
    generatorName.set("typescript-angular")
    inputSpec.set(openApiSpec)
    outputDir.set(layout.buildDirectory.dir("generated/frontend").get().asFile.absolutePath)

    configOptions.set(
            mapOf(
                    "providedInRoot" to "true",
                    "ngVersion" to "21"
            )
    )
}

// Node setup
node {
    version = "20.11.0"
    npmVersion = "10.2.4"
    download = true
    nodeProjectDir.set(file("${project.projectDir}/frontend"))
}
// *******************************
// TASKS
//  Copy generated frontend API → Angular src
val copyFrontendApi = tasks.register<Copy>("copyFrontendApi") {
    dependsOn(generateFrontend)
    from("$buildDir/generated/frontend")
    into("$projectDir/frontend/src/app/api")
}

//  npm install
val npmInstall = tasks.named("npmInstall") {
    dependsOn(copyFrontendApi)
}

// Angular Build
val buildFrontend = tasks.register<NpmTask>("buildFrontend") {
    dependsOn(npmInstall)
    args.set(listOf("run", "build"))
}

//  Copy Angular → Spring Boot static
val copyFrontend = tasks.register<Copy>("copyFrontend") {
    dependsOn(buildFrontend)
    from("$projectDir/frontend/dist/myProject")
    into(layout.buildDirectory.dir("resources/main/static"))
}

//  Backend Build
tasks.named("compileKotlin") {
    dependsOn(generateBackend)
}

// Gesamt-Build - Einstiegspunkt bei ./gradlew build
tasks.named("build") {
    dependsOn(copyFrontend)
}
```

3. Projekt bauen:    
`./gradlew build` // startet den task `tasks.named("build")`

Ablauf:   
```
build
└── copyFrontend
└── buildFrontend
└── npmInstall
└── copyFrontendApi
└── generateFrontend

parallel:
compileKotlin
└── generateBackend
```

Single source of truth:
```
    openapi.yaml
/                   \
↓                    ↓
generateBackend  generateFrontend
```

<p align="right">(<a href="#top">nach oben</a>)</p>

___
## Namenskonvention

Dateien in Kleinbuchstaben mit Bindestrichen definieren, z.B. `user-profile.component.ts` (kebab-case).   
Daraus wird automatisch eine Klasse `UserProfileComponent` generiert. (pascal-case)   
D.h. beim Erstellen einer neuen Component kebab-case verwenden!

kebab-case: `user-profile`   
camelCase: `userProfile`   
PascalCase: `UserProfile`   
snake_case: `user_profile`

<p align="right">(<a href="#top">nach oben</a>)</p>

___
# Architektur & Bootstrapping

1. `index.html` (Einstiegspunkt der Anwendung)
2. `main.ts` wird ausgeführt (Bootstrap der Angular App)
3. AppModule (`app.module.ts`) wird geladen (Hauptmodul der Anwendung) - hier werden imports geladen, provider registriert und die in definierte Component geladen
4. AppComponent (`app.component`) wird geladen (HauptComponent der Anwendung) - hier wird die App-Struktur definiert, z.B. Header, Footer, Router-Outlet)
5. Router-Outlet (`app.routing.ts`) (Platzhalter für die geladenen Components basierend auf der aktuellen Route) wird geladen

```
index.html
↓
main.ts
↓
AppModule - app.module.ts (optional bei Standalone nicht vorhanden)
↓
AppComponent
↓
Router (app.routing.ts) - definiert die Routen und welche Components bei welchen Pfaden geladen werden
↓
Weitere Components / Services
```
<p align="right">(<a href="#top">nach oben</a>)</p>

___
# Wichtige Konzepte

## Components

Eine Component entspricht einem Teil der Benutzeroberfläche, z.B. Header, Footer, Navigation, etc.
Die Component besteht aus vier Teilen:
1) TypeScript-Klasse: Enthält die Logik der Component, z.B. Daten, Methoden, Lifecycle Hooks
2) HTML-Template: Definiert die Struktur und das Layout der Component, z.B. welche Elemente angezeigt werden
3) CSS-Styles: Definieren das Aussehen der Component, z.B. Farben, Abstände, Schriftarten
4) Test-Datei (spec): Enthält Unit-Tests für die Component, um sicherzustellen, dass sie korrekt funktioniert

Neue Component generieren:   
`ng generate component header`

Es entsteht ein neuer Ordner `src/app/header/`   
`export` wird der Klasse automatisch vorangestellt, damit sie woanders importiert werden kann.   
Beispiel: `export class HeaderComponent`

HeaderComponent zur AppComponent hinzufügen:   
In der `app.component.html` folgendes eintragen: `<app-header></app-header>`

<p align="right">(<a href="#top">nach oben</a>)</p>

---
## Service

Ein Service ist eine dependency Injection für Klassen, d.h. Services können von allen Components eingebunden und die Werte verwendet werden. (Serviert Daten an Components)    
In einem Service können auch die HTTP Request ausgeführt werden.   
Ein Service hat die Aufgabe:
- Business Logik aus Komponenten auszulagern
- Daten und Funktionen für mehrere Komponenten bereitzustellen
- Kommunikation mit externen APIs oder Backend-Servern zu ermöglichen
- Zustand der Anwendung zu verwalten
- Utility Funktionen bereitzustellen, z.B. wiederholende Funktionen wie Authentifizierung, Logging, Fehlerbehandlung

Neuen Service generieren:  
`ng generate service friend`

Es entsteht nur eine neue `.ts` Datei
```typescript
import { Injectable } from '@angular/core';
@Injectable({
  providedIn: 'root' // Service wird auf Root-Level bereitgestellt, d.h. überall in der App verfügbar
})
export class FriendService {
  constructor() { }
  getFriends() {
    return ['Anna', 'Max', 'Lisa'];
    // hier könnte auch ein HTTP Request stehen, um die Freunde von einem Server zu holen
  }
}

```

Beispiel: `export class FriendService` injection in Component:
```typescript
import { Component } from '@angular/core';
import { FriendService } from './friend.service';
@Component({
  selector: 'app-friend',
  template: '<p>Friend Component</p>'
})
export class FriendComponent {
  constructor(private friendService: FriendService) {} // Service wird hier injiziert und kann in der Component verwendet werden
  
  getFriends() {
    return this.friendService.getFriends(); // Aufruf der Methode im Service, um die Freunde zu bekommen
  }
}
```
<p align="right">(<a href="#top">nach oben</a>)</p>

___
## RxJS
RxJS in Angular ist ein Werkzeug, um mit Daten zu arbeiten, die später oder mehrfach kommen.   
RxJS sorgt dafür, dass Angular automatisch reagiert, wenn etwas passiert.   
RxJS (Reactive Extensions for JavaScript) ist eine Bibliothek für reaktive Programmierung, die es ermöglicht, asynchrone Datenströme zu erstellen und zu verwalten.
Einfach gesagt: RxJS hilft dabei, mit Daten zu arbeiten, die über Zeit eintreffen (z. B. HTTP-Antworten, User-Events, WebSockets).

Grundidee:   
In RxJS werden Daten als Streams (Datenströme) behandelt. Streams nennt man Observables.
- HTTP-Antworten
- User-Events (z. B. Klicks, Tastatureingaben)
- WebSockets

**Observable – das Kernkonzept**    
Ein Observable ist eine Datenquelle, die mehrere Werte über Zeit senden kann.

Beispiel:
```typescript
const observable = this.http.get('/users'); // Observable
observable.subscribe(data => {
  console.log(data);
});
```
Angular fragt den Server nach den Usern.   
Wenn die Daten ankommen, wird die Funktion in `subscribe()` aufgerufen und die Daten werden geloggt.   
`subscribe()` holt Daten aus dem Observable, bedeutet also: "Wenn Daten kommen, hör zu und mache etwas damit".

Auch in Components, Services, Router und Forms gibt es Observables!   
In Components z.B. um auf Inputs zu reagieren (Forms)
```typescript
this.searchControl.valueChanges.subscribe(value => {
  console.log(value);
});
```
<p align="right">(<a href="#top">nach oben</a>)</p>

___
## Signals
Ein Signal ist ein neues Konzept in Angular, welches eine einfache Möglichkeit bietet, Daten zu speichern, die sich ändern können.   
Wenn sich der Wert eines Signals ändert, aktualisiert Angular automatisch die Oberfläche. Man muss sich also nicht selbst darum kümmern.   
Ein Signal ist eine Funktion, die einen Wert enthält und diesen Wert zurückgibt.
- Signals halten Werte (signal(value)).
- Wenn der Wert sich ändert, aktualisiert Angular automatisch das Template.
- Signals ersetzen oft BehaviorSubject, Observable + async-Pipe in einfachen Use-Cases.


> Signals sind eher für State in Komponenten.   
> RxJS ist eher für Events, Streams und HTTP.

Beispiel Signal in einer Component:
```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <h1>Count: {{ count() }}</h1> <!-- Signal lesen -->
    <button (click)="increment()">+</button>
  `
})
export class CounterComponent {
  count = signal(0); // Signal erstellen

  increment() {
    this.count.update(c => c + 1); // Wert ändern
  }
}
```
In diesem Beispiel wird ein Signal namens `count` erstellt, das den aktuellen Zählerstand hält. Wenn der Button geklickt wird, wird die `increment()`-Methode aufgerufen, die den Wert des Signals aktualisiert. Das Template zeigt automatisch den aktuellen Wert von `count` an, ohne dass manuelle Abonnements oder Change Detection erforderlich sind.  
Signals bieten eine einfache und effiziente Möglichkeit, reaktive Daten in Angular zu verwalten, und können in vielen Fällen die Komplexität von Observables und manueller Change Detection reduzieren.


Beispiel Signal in einem Service:   
```typescript
import { Injectable, signal } from '@angular/core';
@Injectable({
  providedIn: 'root'
})
export class CounterService {
    private _selectedVehicleId = signal<string>('');
    selectedVehicleId = this._selectedVehicleId.asReadonly();

  changeVehicle() {
    this._selectedVehicleId.set("12345")// Beispiel: Fahrzeug-ID aktualisieren
  }
}
```
In diesem Beispiel wird ein Signal namens `_selectedVehicleId` in einem Service erstellt, das die ID des ausgewählten Fahrzeugs hält.    
Die Methode `changeVehicle()` aktualisiert den Wert des Signals, indem sie eine neue ID generiert.   
Das Signal wird als readonly exportiert, damit andere Teile der Anwendung den Wert lesen, aber nicht direkt ändern können.



Lesen und Schreiben von Signals:
- Lesen: `count()` - Ruft den aktuellen Wert des Signals ab.
- Schreiben: `count.set(newValue)` - Setzt den Wert des Signals auf `newValue`.
- Aktualisieren: `count.update(currentValue => newValue)` - Aktualisiert den Wert des Signals basierend auf dem aktuellen Wert, z.B. `count.update(c => c + 1)` erhöht den Zähler um 1.

<p align="right">(<a href="#top">nach oben</a>)</p>

---
## Routing

Routing ermöglicht es, zwischen Components zu navigieren.   
Best practice ist es ein top-Level Modul zu erstellen, um den Router zu erstellen und konfigurieren:

`ng generate module app-routing --flat --module=app`    
(`src/app/app-routing.module.ts` wird erstellt)

1) In `app.module.ts` Import machen: `import { RouterModule, Routes } from '@angular/router';`
2) `import { AppRoutingModule } from './app-routing.module';` wird automatisch in app.module.ts importiert und auch das AppRouterModule
3) Definiere deine Routen in `app-routing.module.ts`: const routes: Routes...
4) Verwende `<router-outlet>` im `app.component.html`
5) Füge die RouterLink-Direktive hinzu `<a routerLink="/home">Home</a>` in den Navigation Component html

<p align="right">(<a href="#top">nach oben</a>)</p>

___
## HTTP Requests
In Angular können HTTP Requests mit dem HttpClient-Modul durchgeführt werden.
``` typescript
import { HttpClient } from '@angular/common/http';

constructor(private http: HttpClient) {}

getUsers() {
  return this.http.get('/api/users'); // Observable zurück
}
```
Aufruf in der Component:
```typescript
this.userService.getUsers().subscribe(users => {
  this.users = users; // automatisch UI aktualisieren
});
```
<p align="right">(<a href="#top">nach oben</a>)</p>

___
## Lazy Loading
Lazy Loading ist eine Technik, bei der bestimmte Teile einer Angular-Anwendung nur dann geladen werden, wenn sie tatsächlich benötigt werden.
Dies kann die Ladezeit der Anwendung verbessern, da nicht alle Komponenten und Module sofort geladen werden müssen.

In Angular spricht man oft von Lazy Loaded Modules:   
Beispiel:
``` typescript
const routes: Routes = [
{ path: '', component: HomeComponent },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
  }
]; 
```
loadChildren sagt Angular:   
„Lade dieses Modul erst, wenn der Nutzer die Route /admin besucht.“   
Das AdminModule wird also nicht sofort beim Start geladen.

<p align="right">(<a href="#top">nach oben</a>)</p>

___
## Forms
In Angular gibt es zwei Hauptansätze für die Arbeit mit Formularen: Template-driven Forms und Reactive Forms.
1. Template-driven Forms:
    - Einfacher Ansatz, bei dem die Formularlogik hauptsächlich im Template definiert wird.
    - Verwendet Direktiven wie `ngModel` für die Datenbindung.
    - Gut geeignet für einfache Formulare mit wenig Logik.

Beispiel:
```html 
<form #myForm="ngForm" (ngSubmit)="onSubmit(myForm)">
   <input name="username" ngModel required>
   <button type="submit">Submit</button>
 </form>
 ```

2. Reactive Forms:
    - Komplexerer Ansatz, bei dem die Formularlogik in der Component definiert wird.
    - Verwendet FormControl, FormGroup und FormBuilder für die Erstellung und Verwaltung von Formularen.
    - Bietet mehr Flexibilität und Kontrolle über die Validierung und den Zustand des Formulars.
    - Gut geeignet für komplexe Formulare mit umfangreicher Logik.

Beispiel:
```typescript
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
export class MyComponent {
  myForm: FormGroup;
 
  constructor(private fb: FormBuilder) {
    this.myForm = this.fb.group({
      username: ['', Validators.required]
    });
  }
  onSubmit() {
    console.log(this.myForm.value);
  }
}
```
<p align="right">(<a href="#top">nach oben</a>)</p>

___
## Testing

Testframeworks:
- Unit Tests: Jasmine + Karma (Standard in Angular)
- End-to-End Tests: Protractor (Standard in Angular, aber wird nicht mehr weiterentwickelt), Cypress (beliebtes alternatives Framework)
- Testen von Services: Testen von Methoden und Logik in Services, z.B. HTTP Requests, Business Logik

Beispiel Service-Test:
```typescript
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { UserService, User } from './user.service';

describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController; // kontrolliert requests

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule], // ersetzt echten HttpClient durch einen Mock
      providers: [UserService]
    });

    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify(); // prüft, dass keine Requests offen sind
  });

  it('should be created', () => {
    expect(service).toBeTruthy();
  });

  it('should fetch users', () => {
    const dummyUsers: User[] = [
      { id: 1, name: 'Anna' },
      { id: 2, name: 'Max' }
    ];

    service.getUsers().subscribe(users => {
      expect(users.length).toBe(2);
      expect(users).toEqual(dummyUsers);
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(dummyUsers); // simuliert Response vom Server
  });
});
```

Beispiel Component-Test (user.component.spec.ts):   
Gegeben:
``` typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-user',
  template: `
    <p>{{ username }}</p>
    <input [(ngModel)]="username" />
    <button (click)="resetUsername()">Reset</button>
  `
})
export class UserComponent {
  username: string = 'Anna';

  resetUsername() {
    this.username = '';
  }
}
```

Test:
```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { FormsModule } from '@angular/forms'; // für ngModel
import { UserComponent } from './user.component';
import { By } from '@angular/platform-browser';

describe('UserComponent', () => {
  let component: UserComponent;
  let fixture: ComponentFixture<UserComponent>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [UserComponent],
      imports: [FormsModule] // ngModel benötigt FormsModule
    }).compileComponents();

    fixture = TestBed.createComponent(UserComponent);
    component = fixture.componentInstance;
    fixture.detectChanges(); // Change Detection starten
  });

  it('should create the component', () => {
    expect(component).toBeTruthy();
  });

  it('should render default username', () => {
    const p: HTMLElement = fixture.nativeElement.querySelector('p');
    expect(p.textContent).toBe('Anna');
  });

  it('should update username when input changes', () => {
    const input = fixture.debugElement.query(By.css('input')).nativeElement;
    input.value = 'Max';
    input.dispatchEvent(new Event('input'));
    fixture.detectChanges();

    const p: HTMLElement = fixture.nativeElement.querySelector('p');
    expect(p.textContent).toBe('Max');
  });

  it('should reset username when button is clicked', () => {
    const button = fixture.debugElement.query(By.css('button')).nativeElement;
    button.click();
    fixture.detectChanges();

    const p: HTMLElement = fixture.nativeElement.querySelector('p');
    expect(p.textContent).toBe('');
  });
});

```
<p align="right">(<a href="#top">nach oben</a>)</p>

___
## Guards
In Angular ist ein Guard eine spezielle Funktion oder Klasse, die verwendet wird, um den Zugriff auf bestimmte Routen zu kontrollieren.   
Guards entscheiden also, ob ein Benutzer eine Route betreten, verlassen oder laden darf.

**AuthGuard:** Zugriff auf bestimmte Routen basierend auf der Authentifizierung steuern   
Der AuthGuard ist ein Service, der die Navigation steuert, basierend auf einer Bedingung (z. B. ob der Benutzer eingeloggt ist oder nicht).

`ng generate guard auth/Auth`   
(Dies erstellt eine Datei namens `auth.guard.ts` im src/app/auth-Verzeichnis.)

AuthService erstellen:    
`ng generate service auth/Auth`

AuthGuard in den Routen verwenden:   
In `app-routing.module.ts` AuthGuard hinzufügen.

Beispiel:
```typescript
export const authGuard: CanActivateFn = (route: ActivatedRouteSnapshot, state: RouterStateSnapshot) => {
    const authService = inject(AuthService);
    const router = inject(Router);
    if (authService.isAuthenticated) {
        return true;
    }
    router.navigate(['/login'], { queryParams: { returnUrl: state.url } });
    return false;
};
```

<p align="right">(<a href="#top">nach oben</a>)</p>

___
## Pipes

In Angular ist eine Pipe eine Funktion, die Daten in Templates transformiert.    
Pipes werden verwendet, um Daten in einer benutzerfreundlichen Weise anzuzeigen, ohne die zugrunde liegenden Daten zu verändern.  
Pipes können in Templates mit der Pipe-Syntax (`|`) verwendet werden. Zum Beispiel:
```html
{{ birthday | date:"longDate" }}
```
In diesem Beispiel wird die `date`-Pipe verwendet, um das `birthday`-Datum in einem langen Datumsformat anzuzeigen.  
Angular bietet viele eingebaute Pipes, wie `date`, `uppercase`, `lowercase`, `currency` und `percent`. Entwickler können auch benutzerdefinierte Pipes erstellen, um spezifische Datenformatierungen zu implementieren, die nicht von den eingebauten Pipes abgedeckt werden.  
Pipes können auch Parameter akzeptieren, um die Transformation weiter anzupassen. Zum Beispiel:
```html
{{ amount | currency:'USD':true }}
```
In diesem Beispiel wird die `currency`-Pipe verwendet, um den `amount`-Wert als Währung anzuzeigen, wobei 'USD' als Währungssymbol verwendet wird und `true` angibt, dass das Währungssymbol angezeigt werden soll.

<p align="right">(<a href="#top">nach oben</a>)</p>

---
## Interceptor

Interceptor kann man als Middleware für HTTP-Anfragen und -Antworten verstehen.  
Intercept heißt so viel wie "abfangen" oder "unterbrechen". Interceptors fangen HTTP-Anfragen und -Antworten ab, bevor sie die Anwendung erreichen oder an den Server gesendet werden.   

In Angular ist ein Interceptor eine spezielle Art von Service, der es ermöglicht, HTTP-Anfragen und -Antworten zu verändern oder zu überwachen.    
Er wird in der Regel verwendet, um bestimmte Operationen global auf alle HTTP-Requests oder -Responses anzuwenden, bevor diese die Anwendung erreichen oder an den Server gesendet werden.    
Interceptors sind Teil des HttpClient-Moduls und bieten eine mächtige Möglichkeit, zusätzliche Logik wie Authentifizierung, Fehlerbehandlung oder das Hinzufügen von Headern hinzuzufügen.

Ein Interceptor implementiert das HttpInterceptor-Interface, dass eine Methode namens `intercept()` bereitstellt. Diese Methode wird immer aufgerufen, wenn eine HTTP-Anfrage gemacht wird.   
Hauptanwendungsfälle für Interceptors:

    Authentifizierung und Autorisierung: Fügen Sie einen Authentifizierungstoken zu den HTTP-Headern hinzu.
    Fehlerbehandlung: Behandeln Sie serverseitige Fehler wie 500 oder 401 und zeigen Sie entsprechende Fehlermeldungen an.
    Logging: Protokollieren Sie Anfragen und Antworten, um Debugging und Analyse zu erleichtern.
    Globale Modifikationen: Fügen Sie z. B. standardmäßig bestimmte Header oder Parameter zu allen Anfragen hinzu.

Erstellen eines Interceptors:   
`ng generate interceptor [name]`

Beispiel eines Interceptors, der einen Authentifizierungstoken hinzufügt:
```typescript
@Injectable()
export const authInterceptor: HttpInterceptorFn = (req, next) => {
    const authToken = 'my-auth-token'; // Hier könnte auch ein Token aus einem AuthService geholt werden

    // Intercept Request: Request klonen und Header setzen
    const authReq = req.clone({
        headers: req.headers.set('Authorization', `Bearer ${authToken}`)
    });

    // Request senden + Response behandeln
    return next(authReq).pipe(
        catchError((error) => {
            if (error?.status === 401) {
                console.log('[AuthInterceptor] 401 → logout');
                auth.logout();

                if (router.url !== '/login') {
                    void router.navigate(['/login'], { replaceUrl: true });
                }
            }
            return throwError(() => error);
        })
    );
}
```

<p align="right">(<a href="#top">nach oben</a>)</p>

___
## Lifecycle Hooks

Die Lifecycle Hooks sind Methoden, die zu bestimmten Zeitpunkten im Leben einer Component oder Direktive automatisch aufgerufen werden.
1. Initialisierung:
    - `constructor`: Wird einmalig aufgerufen, wenn die Component instanziiert wird. Hier sollten keine komplexen Logiken oder Abhängigkeiten verwendet werden. Nur dependency Injection!
    - `ngOnChanges`: Wird aufgerufen, wenn sich @Input()- Werte ändern. Auch beim ersten Setzen der Inputs.
    - `ngOnInit`: Wird einmalig nach der ersten Initialisierung der Component aufgerufen. Beste Stelle für Initialisierungen.
2. Content:
    - `ngDoCheck`: Wird bei jeder Änderungserkennung aufgerufen, um benutzerdefinierte Änderungsprüfungen durchzuführen.
    - `ngAfterContentInit`: Wird einmalig aufgerufen, nachdem der Inhalt der Component (z.B. ng-content) initialisiert wurde.
    - `ngAfterContentChecked`: Wird nach jeder Änderungserkennung aufgerufen, nachdem der Inhalt der Component überprüft wurde.
3. View:
    - `ngAfterViewInit`: Wird einmalig aufgerufen, nachdem die Ansicht der Component (und aller KindComponents) initialisiert wurde.
    - `ngAfterViewChecked`: Wird nach jeder Änderungserkennung aufgerufen, nachdem die Ansicht der Component überprüft wurde.
4. Clean-up:
    - `ngOnDestroy`: Wird einmalig aufgerufen, bevor die Component zerstört wird, z.B. um Ressourcen freizugeben oder Abonnements zu kündigen.

(ngOnChanges + ngDoCheck + After*-Checked wiederholen sich bei Änderungen)

<p align="right">(<a href="#top">nach oben</a>)</p>

___
## Bindings / Directive

**Bindings**   
In Angular gibt es verschiedene Arten von Bindings, die verwendet werden, um Daten zwischen der Component und dem Template zu verbinden. Hier sind die wichtigsten Bindings:
1. **Interpolation {{ }}**: Wird verwendet, um Daten von der Component im Template (nur Text) anzuzeigen. Zum Beispiel: `<h1>{{ title }}</h1>` zeigt den Wert der `title`-Variable an.
2. **Property Binding [ ]**: Wird verwendet, um Eigenschaften von HTML-Elementen oder Components zu binden. Zum Beispiel: `<img [src]="imageUrl">` bindet die `src`-Eigenschaft des `img`-Tags an die `imageUrl`-Variable in der Component. Angular aktualisiert die `src`-Eigenschaft automatisch, wenn sich der Wert von `imageUrl` ändert. Also `imageUrl` ist die Variable in der Component. Wenn die Variable sich in der Component ändert, wird der Wert im DOM automatisch aktualisiert.
3. **Event Binding ( )**: Wird verwendet, um Ereignisse von HTML-Elementen oder Component zu binden. Zum Beispiel: `<button (click)="onClick()">Click me</button>` bindet das `click`-Ereignis des Buttons an die `onClick()`-Methode in der Component.
4. **Two-Way Binding [( )]**: Wird verwendet, um eine bidirektionale Bindung zwischen einer Component und einem Template-Element herzustellen. Zum Beispiel: `<input [(ngModel)]="name">` bindet die `name`-Variable in der Component an das `input`-Element, sodass Änderungen im `input`-Feld automatisch die `name`-Variable aktualisieren und umgekehrt.  Funktioniert nur wenn FormsModule importiert wurde!
5. **Attribute Binding**: Wird verwendet, um HTML-Attribute zu binden. Zum Beispiel: `<button [attr.disabled]="isDisabled ? 'disabled' : null">Click me</button>` bindet das `disabled`-Attribut des Buttons basierend auf dem Wert der `isDisabled`-Variable in der Component.
6. **Class Binding**: Wird verwendet, um CSS-Klassen zu binden. Zum Beispiel: `<div [class.active]="isActive">Content</div>` bindet die `active`-Klasse basierend auf dem Wert der `isActive`-Variable in der Component.
7. **Style Binding**: Wird verwendet, um CSS-Stile zu binden. Zum Beispiel: `<div [style.color]="isError ? 'red' : 'green'">Content</div>` bindet die `color`-Stil basierend auf dem Wert der `isError`-Variable in der Component.

**Direktiven**   
sind spezielle Anweisungen, die das Verhalten von DOM-Elementen oder Components in Angular ändern. Es gibt verschiedene Arten von Direktiven, darunter:
1. **Template Reference Variables**: Werden verwendet, um eine Referenz auf ein DOM-Element oder eine Component im Template zu erstellen. Zum Beispiel: `<input #myInput>` erstellt eine Referenz namens `myInput`, die in der Component verwendet werden kann, um auf das `input`-Element zuzugreifen.
2. **Structural Directives**: Werden verwendet, um die Struktur des DOM basierend auf Bedingungen zu ändern. Zum Beispiel: `@if(isVisible){ <div>Content</div> }` zeigt den `div`-Block nur an, wenn die `isVisible`-Variable in der Component `true` ist. Beispiele:
    1. `@if`, `@else` (ab Angular 21) - neue Syntax für strukturelle Direktiven
    2. `@for` Beispiel: `@for(let item of items){ <div>{{item}}</div> }`
    3. `@switch`


Liste an wichtigen Event-Bindings:
- `(click)`
- `(dblclick)`
- `(input)` - Eingabe in ein Formularfeld verändert sich
- `(keydown)` - Eine Taste wird gedrückt
- `(keyup)` - Eine Taste wird losgelassen
- `(ngSubmit)` - Ein Formular wird abgeschickt ohne page reload (validiert Formular automatisch)

Liste an wichtigen Property-Bindings:
- `[disabled]` - Deaktiviert ein Element
- `[src]` - Quelle eines Bildes
- `[href]` - Link-Adresse
- `[value]` - Wert eines Formularfeldes

> Component → Template: `[value]` (Daten aus der Component ins Template)   
> Template → Component: `(input)` (Daten aus dem Template in die Component)

<p align="right">(<a href="#top">nach oben</a>)</p>

___
## Beziehungen zwischen Components

1. Fall: Kommunikation zwischen Child und Parent Component -> Parent->Child nehme @Input() und Child->Parent nehme @Output()
2. Fall: Kommunikation zwischen Geschwister-Components -> über die gemeinsame Parent-Component oder über einen gemeinsamen Service
3. Fall: Kommunikation zwischen nicht verwandten Components -> Service verwenden

#### @Input() und @Output()
In Angular werden `@Input()` und `@Output()` verwendet, um die Kommunikation zwischen Components zu ermöglichen.
- `@Input()`: Wird verwendet, um Daten von einer übergeordneten Component an eine untergeordnete Component zu übergeben. Es ermöglicht der übergeordneten Component, Werte an die untergeordnete Component zu binden. Zum Beispiel:
```typescript
// child.component.ts
import { Component, Input } from '@angular/core';
@Component({
  selector: 'app-child',
  template: '<p>{{ message }}</p>'
})
export class ChildComponent {
  @Input() message: string; // Deklaration eines Input-Properties
}
// parent.component.html
<app-child [message]="parentMessage"></app-child> <!-- Bindet die parentMessage an das message-Input der ChildComponent -->
```
In diesem Beispiel erhält die `ChildComponent` eine Nachricht von der `ParentComponent` über das `@Input()`-Property `message`. Die `ParentComponent` bindet ihre `parentMessage`-Variable an das `message`-Input der `ChildComponent`.
- `@Output()`: Wird verwendet, um Ereignisse von einer untergeordneten Component an eine übergeordnete Component zu senden. Es ermöglicht der untergeordneten Component, Ereignisse auszulösen, die von der übergeordneten Component behandelt werden können. Zum Beispiel:
```typescript
// child.component.ts
import { Component, Output, EventEmitter } from '@angular/core';
@Component({
  selector: 'app-child',
  template: '<button (click)="sendMessage()">Send Message</button>'
})
export class ChildComponent {
  @Output() messageEvent = new EventEmitter<string>(); // Deklaration eines Output-Events
  sendMessage() {
    this.messageEvent.emit('Hello from Child!'); // Löst das messageEvent mit einem Wert aus
  }
}
// parent.component.html
<app-child (messageEvent)="receiveMessage($event)"></app-child> <!-- Bindet das messageEvent der ChildComponent an die receiveMessage-Methode der ParentComponent -->
<p>{{ receivedMessage }}</p>
```
In diesem Beispiel sendet die `ChildComponent` eine Nachricht an die `ParentComponent`, wenn der Button geklickt wird. Das `@Output()`-Event `messageEvent` wird ausgelöst, und die `ParentComponent` empfängt die Nachricht über die `receiveMessage()`-Methode, die den Wert des Events in der `receivedMessage`-Variable speichert und im Template anzeigt.  
Zusammen ermöglichen `@Input()` und `@Output()` eine effektive Kommunikation zwischen Components in Angular, indem sie Daten und Ereignisse zwischen übergeordneten und untergeordneten Components austauschen.

#### GeschwisterComponents
In Angular können GeschwisterComponents (Sibling Components) auf verschiedene Weise miteinander kommunizieren, da sie sich auf derselben Ebene im Componentsbaum befinden und keine direkte Eltern-Kind-Beziehung haben. Hier sind einige gängige Methoden, um die Kommunikation zwischen GeschwisterComponents zu ermöglichen:
1. **Gemeinsame Parent-Component (nicht bevorzugt)**: Eine Möglichkeit besteht darin, die gemeinsame Parent-Component als Vermittler zu verwenden. Die GeschwisterComponents können Daten oder Ereignisse an die Parent-Component senden, die diese Informationen dann an die andere GeschwisterComponent weiterleitet. Zum Beispiel:
```typescript
// parent.component.ts
import { Component } from '@angular/core';
@Component({
  selector: 'app-parent',
  template: `
    <app-sibling-one (messageEvent)="receiveMessage($event)"></app-sibling-one>
    <app-sibling-two [message]="siblingMessage"></app-sibling-two>
  `
})
export class ParentComponent {
  siblingMessage: string;
  receiveMessage(message: string) {
    this.siblingMessage = message; // Speichert die Nachricht von SiblingOne und gibt sie an SiblingTwo weiter
  }
}
// sibling-one.component.ts
import { Component, Output, EventEmitter } from '@angular/core';
@Component({
  selector: 'app-sibling-one',
  template: '<button (click)="sendMessage()">Send Message to Sibling Two</button>'
})
export class SiblingOneComponent {
  @Output() messageEvent = new EventEmitter<string>();
  sendMessage() {
    this.messageEvent.emit('Hello from Sibling One!'); // Löst das messageEvent mit einer Nachricht aus
  }
}
// sibling-two.component.ts
import { Component, Input } from '@angular/core';
@Component({
  selector: 'app-sibling-two',
  template: '<p>{{ message }}</p>'
})
export class SiblingTwoComponent { 
  @Input() message: string; // Empfängt die Nachricht von der ParentComponent
}
```
In diesem Beispiel sendet `SiblingOneComponent` eine Nachricht an die `ParentComponent`, die diese Nachricht empfängt und an `SiblingTwoComponent` weiterleitet.
2. **Gemeinsamer Service (bevorzugt)**: Eine andere Möglichkeit besteht darin, einen gemeinsamen Service zu verwenden, der von beiden GeschwisterComponents injiziert wird. Der Service kann als zentraler Speicher für Daten oder als Event-Bus fungieren, um Informationen zwischen den GeschwisterComponents auszutauschen. Zum Beispiel:
```typescript
// message.service.ts
import { Injectable } from '@angular/core';
import { Subject } from 'rxjs';
@Injectable({
  providedIn: 'root'
})
export class MessageService {
  private messageSource = new Subject<string>();
  message$ = this.messageSource.asObservable(); // Observable, das von den GeschwisterComponents abonniert werden kann
  sendMessage(message: string) {
    this.messageSource.next(message); // Sendet eine Nachricht an alle Abonnenten
  }
}
// sibling-one.component.ts
import { Component } from '@angular/core';
import { MessageService } from './message.service';
@Component({
  selector: 'app-sibling-one',
  template: '<button (click)="sendMessage()">Send Message to Sibling Two</button>'
})
export class SiblingOneComponent {
  constructor(private messageService: MessageService) {}
  sendMessage() {
    this.messageService.sendMessage('Hello from Sibling One!'); // Sendet eine Nachricht über den MessageService
  }
}
// sibling-two.component.ts
import { Component, OnInit } from '@angular/core';
import { MessageService } from './message.service';
@Component({
  selector: 'app-sibling-two',
  template: '<p>{{ message }}</p>'
})
export class SiblingTwoComponent implements OnInit {
  message: string;
  constructor(private messageService: MessageService) {}
  ngOnInit() {
    this.messageService.message$.subscribe(msg => this.message = msg); // Abonniert das message$ Observable, um Nachrichten zu empfangen
  }
}
```
In diesem Beispiel verwenden beide GeschwisterComponents den `MessageService`, um Nachrichten auszutauschen. `SiblingOneComponent` sendet eine Nachricht über den Service, und `SiblingTwoComponent` empfängt diese Nachricht, indem es das Observable des Services abonniert.  
Diese Methoden ermöglichen eine effektive Kommunikation zwischen GeschwisterComponents in Angular, ohne dass sie direkt miteinander verbunden sein müssen. Die Wahl der Methode hängt von der spezifischen Anwendungsarchitektur und den Anforderungen der Components ab.

<p align="right">(<a href="#top">nach oben</a>)</p>

___
## DTO / Model
In Services kommen durch requests vom Backend oft DTO's zurück.
Um diese Daten in der Angular-Anwendung zu verwenden, werden diese in Models oder Interfaces umgewandelt.

Meist findet man die Models nicht im Service selbst, sondern in einem eigenen Ordner, z.B. `models` oder `interfaces`.

``` typescript
interface User {
  id: number;
  name: string;
  isActive: boolean;
  dateOfBirth: Date;
  orders: Order[];
  descriptions: string[];
}
```
<p align="right">(<a href="#top">nach oben</a>)</p>

___
# Angular CLI (ab Version 21)

Wichtige Befehle in der täglichen Arbeit mit Angular CLI:

| Befehl                                | Shortcut                  | Zweck / Erklärung                              |
| ------------------------------------- | ------------------------- | ---------------------------------------------- |
| `ng new <name>`                       | `ng n <name>`             | Neues Projekt erstellen                        |
| `ng serve`                            | `ng s`                    | Dev-Server starten                             |
| `ng serve -o`                         | –                         | Dev-Server starten + Browser öffnen            |
| `ng serve --configuration production` | –                         | Produktions-Build lokal testen                 |
| `ng build`                            | `ng b`                    | Dev-Build erstellen                            |
| `ng build --configuration production` | –                         | Produktions-Build erstellen                    |
| `ng test`                             | `ng t`                    | Unit-Tests ausführen                           |
| `ng e2e`                              | `ng e`                    | End-to-End Tests ausführen                     |
| `ng lint`                             | –                         | Codequalität prüfen                            |
| `ng update`                           | –                         | Angular & Dependencies aktualisieren           |
| `ng g component <name>`               | `ng g c <name>`           | Neue Component erzeugen                        |
| `ng g service <name>`                 | `ng g s <name>`           | Neuer Service erzeugen                         |
| `ng g directive <name>`               | `ng g d <name>`           | Neue Directive erzeugen                        |
| `ng g pipe <name>`                    | `ng g p <name>`           | Neue Pipe erzeugen                             |
| `ng g module <name>`                  | `ng g m <name>`           | Neues Modul erzeugen                           |
| `ng g guard <name>`                   | `ng g guard <name>`       | Route Guard erzeugen                           |
| `ng g interceptor <name>`             | `ng g interceptor <name>` | HTTP Interceptor erzeugen                      |
| `ng g resolver <name>`                | `ng g resolver <name>`    | Resolver für Pre-Fetch Data erzeugen           |
| `ng g class <name>`                   | `ng g class <name>`       | Neue Klasse erzeugen                           |
| `ng g interface <name>`               | `ng g interface <name>`   | Neue Interface erzeugen                        |
| `ng g enum <name>`                    | `ng g enum <name>`        | Neues Enum erzeugen                            |
| `ng g library <name>`                 | –                         | Neue Library erzeugen                          |
| `ng g web-worker <name>`              | –                         | Web Worker erzeugen                            |
| `ng g service-worker`                 | –                         | Service Worker Setup                           |
| `ng doc <keyword>`                    | –                         | Öffnet die Angular Dokumentation               |
| `ng cache`                            | –                         | CLI Cache verwalten                            |

### Alle Befehle

| Befehl                                | Shortcut | Zweck |
|---------------------------------------|---------|------|
| `ng help`                             | – | Hilfe / Liste aller CLI-Befehle |
| `ng version`                          | `ng v` | Angular CLI & Projektversion |
| `ng new <name>`                       | `ng n` | Neues Projekt erstellen |
| `ng add <package>`                    | – | Package hinzufügen & konfigurieren |
| `ng update`                           | – | Angular & Dependencies aktualisieren |
| `ng config`                           | – | `angular.json` konfigurieren |
| `ng analytics`                        | – | CLI Analytics konfigurieren |
| `ng cache`                            | – | CLI Cache verwalten |
| `ng completion`                       | – | Autocompletion einrichten |
| `ng doc <keyword>`                    | – | Dokumentation öffnen |
| `ng deploy`                           | – | Deployment ausführen |
| `ng run <target>`                     | – | Custom Builder ausführen |
| `ng extract-i18n`                     | – | i18n Nachrichten extrahieren |
| `ng build`                            | `ng b` | Dev-Build erstellen |
| `ng build --configuration production` | – | Produktions-Build |
| `ng serve`                            | `ng s` | Dev-Server starten |
| `ng serve -o`                         | – | Server + Browser öffnen |
| `ng serve --configuration production` | – | Produktions-Build lokal testen |
| `ng test`                             | `ng t` | Unit-Tests ausführen |
| `ng e2e`                              | `ng e` | End-to-End-Tests |
| `ng lint`                             | – | Codequalität prüfen |
| `ng g application <name>`             | – | Neue Anwendung |
| `ng g component <name>`               | `ng g c` | Neue Component |
| `ng g directive <name>`               | `ng g d` | Neue Directive |
| `ng g pipe <name>`                    | `ng g p` | Neue Pipe |
| `ng g guard <name>`                   | `ng g guard` | Route-Guard |
| `ng g interceptor <name>`             | `ng g interceptor` | HTTP-Interceptor |
| `ng g resolver <name>`                | `ng g resolver` | Resolver (Pre-Fetch Data) |
| `ng g module <name>`                  | `ng g m` | Neues Modul |
| `ng g service <name>`                 | `ng g s` | Neuer Service |
| `ng g class <name>`                   | `ng g class` | Neue Klasse |
| `ng g interface <name>`               | `ng g interface` | Neue Interface |
| `ng g enum <name>`                    | `ng g enum` | Neues Enum |
| `ng g environments`                   | – | Environment-Dateien erzeugen |
| `ng g config <name>`                  | – | CLI-Konfig-Dateien erzeugen |
| `ng g library <name>`                 | – | Neue Library |
| `ng g service-worker`                 | – | Service Worker Setup |
| `ng g web-worker <name>`              | – | Web Worker erzeugen |

<p align="right">(<a href="#top">nach oben</a>)</p>
