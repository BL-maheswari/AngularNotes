# AngularNotes



explain the observables working in angular 

Observables are one of the most important concepts in Angular because Angular uses them heavily for HTTP requests, routing, forms, events, and services.

Think of an Observable as a data stream that can emit values over time.

What is an Observable?
An Observable is an object provided by the RxJS library that produces data asynchronously.

Imagine a YouTube live stream:

The channel = Observable

Viewers = Subscribers

New videos/messages = Emitted values

Until someone subscribes, the Observable usually does nothing.

import { Observable } from 'rxjs';

const observable = new Observable(observer => {
  observer.next('Hello');
  observer.next('Angular');
  observer.complete();
});
Here:

observer.next()
sends data to subscribers.

observer.complete()
indicates the stream is finished.

Observable Lifecycle
There are three stages:

1. Creation
const observable = new Observable(observer => {
  observer.next('Data');
});
2. Subscription
observable.subscribe(data => {
  console.log(data);
});
Output:

Data
Without subscribe(), nothing happens.

3. Completion / Error
const observable = new Observable(observer => {
  observer.next('Success');
  observer.complete();
});
or

const observable = new Observable(observer => {
  observer.error('Something went wrong');
});
How Angular Uses Observables
Example 1: HTTP Requests
When Angular makes an API call:

this.http.get('/api/books')
Angular returns:

Observable<any>
Not actual data.

books$: Observable<any>;
The data comes later when the server responds.

Flow
Component
    |
    |
    V
HttpClient.get()
    |
    |
Returns Observable
    |
    |
subscribe()
    |
    |
Receives Response
Example:

this.http.get('/api/books')
  .subscribe(response => {
      console.log(response);
  });
Internal Working of HTTP Observable
When Angular sees:

this.http.get('/api/books')
it creates an Observable.

Observable Created
       |
       |
No Request Yet
Only after:

.subscribe()
Angular actually sends the request.

subscribe()
      |
      V
HTTP Request Sent
      |
      V
Server Response
      |
      V
observer.next(response)
      |
      V
observer.complete()
Why Not Return Data Directly?
Consider:

getBooks() {
   return this.http.get('/api/books');
}
The server may take:

1 second

5 seconds

10 seconds

Angular cannot wait and block the UI.

So it returns an Observable immediately and continues executing other code.

When the response arrives, Angular notifies subscribers.

Observable vs Function
Function
function getData() {
   return "Hello";
}
Output:

Call Function
      |
      V
Return Value
      |
      V
End
Only one value.

Observable
observable.subscribe();
Output:

Start Stream
      |
      V
Value 1
      |
      V
Value 2
      |
      V
Value 3
      |
      V
Complete
Multiple values can be emitted.

Observable vs Promise
Observable	Promise
Multiple values	Single value
Lazy execution	Executes immediately
Can be cancelled	Cannot be cancelled easily
RxJS operators available	Limited operators
Used heavily in Angular	Less used
Example Promise:

fetch('/api/books')
  .then(res => res.json());
One response only.

Example Observable:

interval(1000)
Can emit forever.

1
2
3
4
5
...
Subscribe Method
Most Angular developers use:

this.bookService.getBooks()
  .subscribe({
    next: (data) => {
      console.log(data);
    },
    error: (err) => {
      console.log(err);
    },
    complete: () => {
      console.log('Completed');
    }
  });
next()
Receives data.

next: data => console.log(data)
error()
Receives errors.

error: err => console.log(err)
complete()
Runs when Observable finishes.

complete: () => console.log('Done')
Operators
RxJS provides operators to transform data.

Example:

import { map } from 'rxjs/operators';

this.http.get('/api/books')
  .pipe(
    map((data: any) => data.result)
  )
  .subscribe(res => {
    console.log(res);
  });
Flow:

Observable
     |
     V
pipe()
     |
     V
map()
     |
     V
subscribe()
Subject (Special Observable)
A Subject is both:

Observable

Observer

import { Subject } from 'rxjs';

const subject = new Subject();
Send data:

subject.next('Hello');
Receive data:

subject.subscribe(data => {
   console.log(data);
});
Real Angular Service Example
Service
@Injectable({
  providedIn: 'root'
})
export class BookService {

  constructor(
    private http: HttpClient
  ) {}

  getBooks() {
    return this.http.get(
      'https://api.example.com/books'
    );
  }
}
Component
export class BookComponent {

  books: any[] = [];

  constructor(
    private bookService: BookService
  ) {}

  ngOnInit() {

    this.bookService.getBooks()
      .subscribe({
        next: (response: any) => {
          this.books = response.result;
        },
        error: (err) => {
          console.log(err);
        }
      });

  }
}
Java Comparison
Since you know Java, think of an Observable like a combination of:

interface Observer {
    void update(String data);
}
and

class Publisher {
    List<Observer> observers;

    void notifyObservers() {
        // send data
    }
}
Angular Observable follows the same Publisher → Subscriber pattern:

Service (Publisher)
        |
        V
Observable
        |
        V
Component (Subscriber)
Example:

publisher.notifyObservers("Book Added");
Equivalent in Angular:

subject.next("Book Added");
Memory Visualization
Component Created
       |
       V
Subscribe to Observable
       |
       V
Observable stored in memory
       |
       V
Receives values continuously
       |
       V
Unsubscribe OR Complete
       |
       V
Memory Released
For long-running streams (events, intervals, Subjects), forgetting to unsubscribe can cause memory leaks. That's why Angular developers often unsubscribe in ngOnDestroy() or use the async pipe.

In one sentence: An Observable in Angular is a lazy asynchronous data stream that produces values over time, and components receive those values by subscribing to the stream.


