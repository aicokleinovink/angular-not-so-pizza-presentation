
class: center, middle

# Angular (not so) Pizza Avond

---

# Agenda

1. Compound Components
2. Angular CLI & Angular Playground
3. Demo: TabGroupComponent
4. Exercise: ChartComponent
5. Discussion: Services, Architecture, RxJS...

---

# Compound Components

Creating a `SliderComponent`, which takes some `slides` and a slide `speed`:
```html
<slider [slides]="mySlides" [speed]="4000"></slider>
```

---

# Compound Components

Let's add `width` and `height` to our slider:
```html
<slider 
    [slides]="mySlides" 
    [speed]="4000"
    [width]="800"
    [height]="600">
</slider>
```

---

# Compound Components

But we need even more!
```html
<slider 
    [slides]="mySlides" 
    [speed]="4000"
    [width]="800"
    [height]="600"
    [navigation]="true"
    [disabled]="false"
    (slideSelected)="slideSelected($event)">
</slider>
```

---

# Compound Components

Let's create a compound component! :-)
```html
<slider [disabled]="false">

  <slider-rotator [speed]="4000" [width]="800" [height]="600"> 
    <slider-slide 
        *ngFor="let slide of slides" 
        [slide]="slide" 
        (selected)="slideSelected(slide)">
    </slider-slide>
  </slider-rotator>
  
  <slider-navigation></slider-navigation>
</slider>
```
---
class: center, middle

# Angular Playground

#### "An open source tool for building enterprise Angular components, directives, and pipes in isolation"

http://www.angularplayground.it/

---

# Angular Playground

A simple compound `TabGroupComponent`:
```html
<app-tab-group>
  <app-tab label="Tab 1"> Show when Tab 1 is active </app-tab>
  <app-tab label="Tab 2"> Show when Tab 2 is active </app-tab>
</app-tab-group>
```

Create a new Angular project with the `Angular CLI`:
```yaml
$ npm install -g @angular/cli
$ ng new not-so-pizza-project
$ cd not-so-pizza-project && ng add angular-playground
```

Add a new `TabComponent` to your project:
```yaml
$ ng g component components/tab
$ ng g angular-playground:sandbox components/tab tab 
```

After fixing import in `tab.component.sandbox.ts` run:

```yaml
$ npm run playground tab
```

---

# Code: TabComponent
```typescript
@Component({
  selector: 'app-tab',
  templateUrl: 'tab.component.html',
})
export class TabComponent {
  @Input()
  public label: string;

  @Input()
  public active: boolean;

  constructor() {
    this.label = null;
    this.active = false;
  }
}
``` 

```html
<div class="app-tab">
  <ng-container *ngIf="active">
    <ng-content></ng-content>
  </ng-container>
</div>
```

---

# Code: TabGroupComponent
```typescript
@Component({
  selector: 'app-tab-group',
  templateUrl: 'tab-group.component.html',
})
export class TabGroupComponent implements AfterContentInit {
  @ContentChildren(TabComponent) 
  public tabs: QueryList<TabComponent>;
  
  constructor() {
    this.tabs = null;
  }
  
  public ngAfterContentInit(): void {
    this.setFirstTabActive();
  }
  
  public onClick(tabIndex: number): void {
    this.tabs.map((tab, index) => tab.active = index === tabIndex);
  }
  
  private setFirstTabActive(): void {
    const activeTabs = this.tabs.toArray().filter(tab => tab.active);
    if (activeTabs.length === 0) {
      this.tabs.first.active = true;
    }
  }
}
```

---

# Code: TabGroupComponent
```html
<div class="app-tab-group">

  <ul class="app-tab-group__tabs">
    <li *ngFor="let tab of tabs; let i = index"
        class="app-tab-group__tab"
        [class.app-tab-group__tab--active]="tab.active">
      <a (click)="onClick(i)"> {{ tab.label }} </a>
    </li>
  </ul>
  
  <div class="app-tab-group__content">
    <ng-content></ng-content>
  </div>

</div>
```

---

# Code: Sandbox TabGroupComponent
```typescript
export default sandboxOf(TabGroupComponent, { declarations: [TabComponent] })
  .add('default', {
    template: `
      <app-tab-group>
        <app-tab *ngFor="let tab of tabs" [label]="tab.label">
          {{ tab.content }}
        </app-tab>
      </app-tab-group>
    `,
    context: {
      tabs: [
        { label: 'Tab 1', content: 'Content Tab 1' },
        { label: 'Tab 2', content: 'Content Tab 2' },
      ],
    },
  });
```

---

# Exercise: Compound Charts Component

Install dependencies:
```yaml
$ npm install ng2-charts chart.js
``` 
In `app.module.ts` add the `ChartsModule` and import `chart.js`:
```typescript
import 'chart.js';
import { ChartsModule } from 'ng2-charts';

// in your App Module
imports: [
  ...,
  ChartsModule
]
```
Generate components and sandbox:
```yaml
$ ng g component components/someComponent
$ ng g angular-playground:sandbox components/some-component someComponent 
$ npm run playground someComponent
```
https://valor-software.com/ng2-charts/

---

# Exercise: Compound Charts Component

For example:
```html
<app-chart [legend]="false" type="line">

  <app-chart-labels 
      labels="['a', 'b', 'c', 'd', 'e']">
  </app-chart-labels>
  
  <app-chart-data 
      label="Data Set 1"
      data="[100, 120, 145, 140, 160]">
  </app-chart-data>
  
  <app-chart-data 
      label="Data Set 2"
      data="[110, 115, 155, 165, 180]">
  </app-chart-data>
  
</app-chart>
```

__Sandboxing__

Don't forget to import `ChartsModule` in your sandbox!

---

# Code: ChartDataComponent
```typescript
@Component({
  selector: 'app-chart-data',
  template: '',
})
export class ChartDataComponent implements OnInit {
  @Input()
  public data: number[];

  @Input()
  public label: string;

  constructor(private chart: ChartComponent) {
    this.data = [];
    this.label = null;
  }

  public ngOnInit(): void {
    this.chart.createDataSet(this.data, this.label);
  }
}
```

---

# Code: ChartLabelsComponent
```typescript
@Component({
  selector: 'app-chart-labels',
  template: '',
})
export class ChartLabelsComponent implements OnInit {
  @Input()
  public labels: string[];

  constructor(private chart: ChartComponent) {
    this.labels = [];
  }

  public ngOnInit(): void {
    this.chart.setLabels(this.labels);
  }
}
```

---

# Code: ChartComponent
```html
<div class="app-chart"> <!-- container is needed! -->
  <canvas
      baseChart
      [datasets]="dataSets"
      [labels]="labels"
      [options]="options"
      [chartType]="type"
      [legend]="legend">
  </canvas>
</div>
```

---

# Code: ChartComponent
```typescript
@Component({
  selector: 'app-chart',
  templateUrl: './chart.component.html',
})
export class ChartComponent {
  @Input()
  public type: string;

  @Input()
  public legend: boolean;

  public dataSets: ChartDataSet[];
  public labels: string[];
  public options: any;
  private dataSetsMap: Map<string, ChartDataSet>;

  constructor() {
    this.type = 'line';
    this.legend = false;
    this.dataSets = [];
    this.labels = [];
    this.options = { responsive: true };
    this.dataSetsMap = new Map();
  }

  ...
```

---

# Code: ChartComponent
```typescript
  ...

  public createDataSet(data: number[], label: string): void {
    if (!this.dataSetsMap.get(label)) {
      this.dataSetsMap.set(label, { label, data });
    }
    this.dataSets = Array.from(this.dataSetsMap.values());
  }

  public setLabels(labels: string[]): void {
    this.labels = labels;
  }
}
```

```typescript
export interface ChartDataSet {
  label: string;
  data: number[];
}
```

---

# Discussion: Angular Services
```typescript
@Injectable()
export class UserService {
  private user: BehaviorSubject<User>;
  
  constructor(private afs: AngularFirestore) {
    this.user = new BehaviorSubject<User>(null);
  }
  
  public get user$(): Observable<User> {
    return this.user.asObservable();
  }
  
  public loadUser(userId: string): void {
    this.afs.doc(`users/${userId}`)
      .valueChanges()
      .subscribe(data => this.user.next(data));
  }
}
```
---

# Discussion: Angular Services
```typescript
@Component({
  selector: 'app-some-page',
  template: `
    <some-component [user]="user$ | async"></some-component>
  `
})
export class SomePage implements OnInit {
  public user$: Observable<User>;
  
  constructor(private userService: UserService) {
    this.user$ = null;
  }
  
  public ngOnInit(): void {
    this.user$ = this.userService.user$.pipe(
      map(user => {
        if (user) {
          // do stuff
        }
        return user;
      })
    );
    
    this.userService.loadUser('some-user-id');
  }
}
```
---

### Discussion: App Structure
```yaml
- app
  |-- core
  |   |-- authentication
  |   |   |-- index.ts
  |   |   |-- authentication.service.ts
  |   |   |-- authentication.service.spec.ts
  |   |-- [+] user
  |   |-- core.module.ts (> app.module.ts)
  |-- public
  |   |-- pages
  |   |   |-- home
  |   |   |   |-- index.ts
  |   |   |   |-- home.component.ts
  |   |   |   |-- home.component.spec.ts
  |   |   |   |-- home.component.scss
  |   |   |   |-- home.component.html
  |   |   |-- [+] singup
  |   |   |-- [+] signin
  |   |-- [+] components
  |   |-- [+] services
  |   |-- [+] pipes  
  |   |-- [+] ...
  |   |-- public.module.ts (lazy)
  |-- admin
  |   [+] ...
  |   |-- admin.module.ts (lazy)
  |-- app.module.ts
```
---
class: center, middle
# Discussion: RxJS

Can somebody please write documentation for the documentation?!
---
# Cool other stuff

- Prettier - opinionated code formatter (https://prettier.io/)
- Husky - git hooks made easy (https://github.com/typicode/husky)