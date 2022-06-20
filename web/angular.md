# Angular Tutorials

## Accept Input Property
1. Add import:
	```ts
		import { ... Input ... } from '@angular/core';
	```
2. Add to component class:
	```ts
		@Input() page = '/default';
	```
	
3. Use:
	```html
		<app-component [page]="id"></app-component>
	```

## Accept Query Parameter
1. Import ActivatedRoute (in component)
	```ts
		import { ActivatedRoute } from '@angular/router';
	```
	
2. Add to component class:
	```ts
		...
		  id: string | undefined;
	  
		  constructor(private route: ActivatedRoute,) { }

		  ngOnInit() {
		  
			this.route.queryParams
			  .subscribe(params => {
				//console.log(params); // { orderby: "price" }
				//console.log(this.id); // price
				this.id = params['id'];
			  }
			);
		...
	```
	
3. Add to *.component.html
	```html
		<ng-container *ngIf="id">
			...
				<app-item [itemId]="id"></app-comments>
			...
		</ng-container>
	```

## Use Firebase in Angular
1. Install AngularFire for Angular
	```sh
		ng add @angular/fire
	```
2. 	Running the above will automatically add the proper imports to app.module.ts, as well as creating environment config for the application.
3. Use it as follows, in any component:
	```ts
		import { Firestore, collectionData, collection } from '@angular/fire/firestore';
		import { Observable } from 'rxjs';

		interface Item {
		  name: string,
		  ...
		};

		@Component({
		  selector: 'app-root',
		  template: `
		  <ul>
			<li *ngFor="let item of item$ | async">
			  {{ item.name }}
			</li>
		  </ul>
		  `
		})
		export class AppComponent {
		  item$: Observable<Item[]>;
		  constructor(firestore: Firestore) {
			const collection = collection(firestore, 'items');
			this.item$ = collectionData(collection);
		  }
		}	
	```
4. You may also need to update the rules for your Firestore DB:
	```
		rules_version = '2';
		service cloud.firestore {
		  match /databases/{database}/documents {
			match /{document=**} {
			  allow read;
			  allow write: if request.auth.uid != null;
			}
		  }
		}
	```
### Sources
- [AngularFire docs](https://www.npmjs.com/package/@angular/fire)

## Use Sanity in Angular
1. Install Sanity Client & Image URL Builder
	```sh
		npm install @sanity/client @sanity/image-url @ng-web-apis/common
	```
	
2. Add config options to environment.ts
	```ts
		export const environment = {
		  production: false,
		  sanity: {
			projectId: '< sanity project id>',
			dataset: '< dataset, usually production|development >',
			useCdn: true,
		  },
		  web: {
			url: '<#< deployments.web.url >#>',
		  },
		}
	```
	
3. Update typescript config files
	```json
		// tsconfig.json
		"compilerOptions": { 
		  "allowSyntheticDefaultImports": true,
		  ...
		},
		// tsconfig.app.json
		"compilerOptions": {
		  "types": ["node"],
		  ...
		},
	```
4. Create Sanity Service
	```sh
		ng generate service SanityService
	```
	
	```ts
		import { HttpClient } from '@angular/common/http';
		import { Inject, Injectable } from '@angular/core';
		import { WINDOW } from '@ng-web-apis/common';
		import sanityClient, { ClientConfig, SanityClient } from '@sanity/client';
		import imageUrlBuilder from '@sanity/image-url';
		import { ImageUrlBuilder } from '@sanity/image-url/lib/types/builder';
		import { SanityImageSource } from '@sanity/image-url/lib/types/types';
		import { map, Observable } from 'rxjs';
		import { environment } from '../environments/environment';

		@Injectable({
			providedIn: 'root',
		})
		export class SanityService {
			private client: SanityClient;
			private imageUrlBuilder: ImageUrlBuilder;
			private clientConfig: ClientConfig = {
				projectId: environment.sanity.projectId,
				dataset: environment.sanity.dataset,
				apiVersion: this.getApiVersion(),
				useCdn: false,
			};

			constructor(private http: HttpClient, @Inject(WINDOW) private wnd: Window,) {
				this.client = this.createClient();
				this.imageUrlBuilder = imageUrlBuilder(this.client);
			}

			getImageUrlBuilder(source: SanityImageSource): ImageUrlBuilder {
				return this.imageUrlBuilder.image(source);
			}

			fetch<T>(query: string): Observable<T> {
				const url = `${this.generateSanityUrl()}${encodeURIComponent(query)}`
				return this.http.get(url).pipe(map((response: any) => response.result as T));
			}

			private createClient(): SanityClient {
				return sanityClient(this.clientConfig);
			}

			private generateSanityUrl(): string {
				const { apiVersion, dataset, projectId } = this.clientConfig;

				const baseUrl = this.wnd.location.href.startsWith(environment.web.url) || this.wnd.location.href.startsWith('http://localhost:4200') ? `https://${projectId}.api.sanity.io/` : `${this.wnd.location.origin}/api/`;

				return `${baseUrl}${apiVersion}/data/query/${dataset}?query=`;
			}

			private getApiVersion(): string {
				return `v${new Date().toISOString().split('T')[0]}`;
			}
		}
	```
	
5. Create Angular Pipe 
	- Create Sanity-Image-Pipe (sanity-image.pipe.ts):
	```ts
		import { Pipe, PipeTransform } from '@angular/core';
		import { SanityService } from './sanity-service.service';
		import { SanityImageSource } from '@sanity/image-url/lib/types/types';

		@Pipe({ name: 'sanityImage' })
		export class SanityImagePipe implements PipeTransform {
			constructor(private sanityService: SanityService) {}

			transform(value: SanityImageSource, width?: number): string {
				if (width) {
					return this.sanityService.getImageUrlBuilder(value).width(width).url();
				}
				return this.sanityService.getImageUrlBuilder(value).url();
			}
		}
	```
	- Then, insert the pipe into the declarations (in app.module.ts):
	```ts
		@NgModule({
			declarations: [
				...
				SanityImagePipe,
				...
			]
		})
	```
6. Import in your application:
	```ts
		import { Observable } from 'rxjs';
		import { SanityService } from '../sanity-service.service';
	```
7. Use in your application
	Example component:
	
	```ts
		export interface Movie {
			_id: String,
			title: String,
			releaseDate: Date,
			body: String,
			poster: SanityImageSource, // = string | SanityReference | SanityAsset | SanityImageObject etc.
		}

		@Component({
		  selector: 'app-movies',
		  template: `
			<main class="container">
			  <div *ngFor="let movie of movies$ | async">
				<img [src]="movie.poster | sanityImage:200" />
				<h3>{{ movie.title }}</h3>
				<p>Released on {{ movie.releaseDate | date }}</p>
			  </div>
			</main>
		  `,
		  styleUrls: ['./movies.component.css']
		})
		export class MoviesComponent implements OnInit {
		  movies$: Observable<Movie[]>
			
		  constructor(private sanityService: SanityService ) { 
			 this.movies$ = this.sanityService.fetch<Movie[]>(
			  `*[_type == "movie"]{
				_id,
				title,
				releaseDate,
				poster,
				author->
			  }`
			 )  
		  }
		}
	```
	* Note: "author->" creates a join to an "Author" table/interface
	
8. Make HttpClient available to all modules
	Add import to app.module.ts:
	```ts
		import { HttpClientModule } from '@angular/common/http';
	```
	
	Add import to NgModule decorator:
	```ts
	@NgModule({
		...
		imports:[ HttpClientModule ]
		...
	})
	```

9. Create "ToHTMLPipe" to convert block content into HTML
	- First, install to-html package
	```sh
		npm i @portabletext/to-html
	```
	- Next, create to-html.pipe.ts
	```ts
		import { Pipe, PipeTransform } from '@angular/core';
		import {toHTML} from '@portabletext/to-html';

		@Pipe({ name: 'toHTML' })
		export class ToHTMLPipe implements PipeTransform {
			constructor() {}

			transform(value: any): string {
				return toHTML(value);
			}
		}
	```
	- Then, insert the pipe into the declarations (in app.module.ts)
	```ts
		@NgModule({
			declarations: [
				...
				ToHTMLPipe,
				...
			]
		})
	```
	- Finally, use it as follows in your component.html file:
	```html
		...
		    <div innerHTML="{{ post.body | toHTML }}" class="text-coolGray-800 p-10">
				<!--
					Content Will Go Here
				-->
			</div>
		...
	```
	
### Sources
- [General Tutorial](https://enlear.academy/use-sanity-cms-in-angular-application-7662c9627250)
- [HttpClient](https://stackoverflow.com/questions/47236963/no-provider-for-httpclient)
- [portabletext/to-html Package](https://github.com/portabletext/to-html)
- [Bind Raw HTML in Angular](https://stackoverflow.com/questions/34585453/how-to-bind-raw-html-in-angular2)
- [Using Pipes](https://angular.io/guide/pipes)


## Format Dates in Angular
1. Install dayjs
	```sh
		npm install dayjs
	```

2. Create formate-date pipe (format-date.pipe.ts)
	```ts
		import { Pipe, PipeTransform } from '@angular/core';
		import * as dayjs from 'dayjs';

		@Pipe({ name: 'formatDate' })
		export class FormatDatePipe implements PipeTransform {
			constructor() {}

			transform(value: any): string {
				return dayjs(value).format('DD/MM/YYYY');
			}
		}
	```

3. Insert the pipe into the declarations (in app.module.ts)
	```ts
		@NgModule({
			declarations: [
				...
				FormatDatePipe,
				...
			]
		})
	```
	
4. Use in component.html like so:
	```html
		...
			<time class="text-sm text-gray-600">
                {{ post.publishedAt | formatDate }}
            </time>
		...
	```

### Sources
- [DayJS](https://day.js.org/en/)
- [Using Pipes](https://angular.io/guide/pipes)
