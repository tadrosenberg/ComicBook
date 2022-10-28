# Tutorial

This tutorial will walk help you build a website for browsing the [xkcd comic](https://xkcd.com/).

## Loading data from an API

The first step is to load data from the XKCD API. We've done this with fetch in JavaScript before, but this time we're going to use the [axios](https://github.com/axios/axios) library. This is a popular library for sending AJAX requests.

One complication is that the [official XKCD API](https://xkcd.com/json.html) does not use CORS, so this will result in a CORS error. To avoid this, we'll use an [unofficial, but CORS-enabled XKCD API](https://github.com/mrmartineau/xkcd-api).

The first step is to create `index.html` to setup React and fetch the latest XKCD comic:

```
<!DOCTYPE html>
<html lang="en">

    <head>
        <script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
        <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script>
        <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
        <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    </head>

    <body>
        <div id="root"></div>
        <script type="text/babel">
        class Xkcd extends React.Component {
            constructor(props) {
                super(props);
                this.state = {
                    comicNumber: 'latest',
                    lastComic: 0,
                    current: {
                      title: '',
                      img: '',
                      alt: ''
                    },
                };
        
                this.getXKCD = this.getXKCD.bind(this);
                this.prev = this.prev.bind(this);
                this.getXKCD();
            }
        
            getXKCD() {
                var url = 'https://xkcd.vercel.app/?comic=' + this.state.comicNumber;
                axios.get(url)
                .then(response => {
                  let comicNum = response.data.num;
                  this.state.comicNumber = comicNum;
                  if(this.state.lastComic == 0) {
                      this.state.lastComic = comicNum;
                  }
                  console.log(this.state);
                  // Since we render current, we need to call setState()
                  this.setState(prevState => {
                    // creating copy of state variable current
                    let current = Object.assign({}, prevState.current);  
                    current = response.data;    // update the object
                    return { current };         // return new object current
                  });
                  return true;
                })
                .catch(error => {
                  console.log(error)
                });
            }
            prev() {
                this.state.comicNumber = this.state.comicNumber - 1;
                this.getXKCD();
            }
        
            render() {
                return (
                    <div>
                        <h1> XKCD Comics </h1>
                        <div>
                            <h2>{this.state.current.safe_title}</h2>
                            <img src={this.state.current.img} alt={this.state.current.alt}></img>
                            <p>{this.state.current.alt}</p>
                            <button onClick={this.prev}>Previous</button>
                        </div>
                    </div>
                );
            }
        }
    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<Xkcd />);
    </script>
    </body>

</html>
```

You should be familiar with most of this from our prior React tutorials. We create a React Component and render it in the "root" div.

We also create a method, called `getXKCD()` to fetch the comic. This method uses the axios API and JavaScript Promises to fetch the comic.

The important thing to know here is that `axios.get` returns a Promise, which gets resolved when the fetch completes. When it does, the code you place in `then` gets executed. This is what it means to have your JavaScript execute asynchronously. Anything placed after `axios.get` (after the final semicolon) is executed immediately after the call to `get`. The code inside `then` is executed only after the resource is actually fetched.

Once the `get` returns, we execute a function inside of `then`. This function takes one argument, `response` that holds the javascript object returned by the API

If you prefer, you can instead use the `async/await` syntax. You could change your `getXKCD` function to look like this:

```
             async getXKCD() {
	       try {
                var url = 'https://xkcd.vercel.app/?comic=' + this.state.comicNumber;
                const response = axios.get(url);
                let comicNum = response.data.num;
                this.state.comicNumber = comicNum;
          	...
                } catch(error) {
                  console.log(error);
                }
            }
```

In this case, the browser waits until the `get` call returns, then stores the result in `response`.  This new syntax is supported by newer browsers. In future projects, we will show you how to package your code so that it will run on any browser.

Here's an example of what the JSON looks like that is returned from the API:

```
{
  "month":"2",
  "num":1958,
  "link":"",
  "year":"2018",
  "news":"",
  "safe_title":"Self-Driving Issues",
  "transcript":"",
  "alt":"If most people turn into muderers all of a sudden, we'll need to push out a firmware update or something.",
  "img":"https://imgs.xkcd.com/comics/self_driving_issues.png",
  "title":"Self-Driving Issues",
  "day":"21",
  "imgRetina":"https://imgs.xkcd.com/comics/self_driving_issues_2x.png"
 }
```

We use this JSON data by putting the following code inside of the render()

```
                        <h1> XKCD Comics </h1>
                        <div>
                            <h2>{this.state.current.safe_title}</h2>
                            <img src={this.state.current.img} alt={this.state.current.alt}></img>
                            <p>{this.state.current.alt}</p>
                            <button onClick={this.prev}>Previous</button>
                        </div>
```

This will bind the `safe_title`, `src` and `alt` properties of the React state to some HTML elements.

## Saving the max comic number
You will need to add buttons to allow you to see the first, last, previous, next and random comics.  
Notice that on the first time getXKCD() is called (when the state variable lastComic is zero), we save the current comic number to lastComic.
This will allow you to use this later when you implement a button to go to the last comic and check to make sure that your next button does
not go past the end of available comics.

## Showing additional information about the comic

We should show some additional information about the comic. We can do this inside of the render() function

```
      <p><i>#{this.state.current.num}, drawn on {this.state.current.month}-{this.state.current.day}-{this.state.current.year}</i></p>
```

## Navigating with buttons

We would like to be able to navigate to comics written on other days. To do this, add some buttons to `index.html`:
We have provided you with the button for Previous.  Add Next, First and Last.

## Navigating to a random comic

We'll now add another button to navigate to a random comic. To do this, add the button with the other buttons
in `index.html`:

You will want to generate a random number between 1 and the maximum this.state.lastComic

```
    getRandom(min, max) {
      min = Math.ceil(min);
      max = Math.floor(max);
      return Math.floor(Math.random() * (max - min + 1)) + min; //The maximum and minimum are inclusive
    }
    randomComic() {
      this.state.comicNumber = this.getRandom(1, this.state.lastComic);
      this.getXKCD();
    }
```

This will choose a random integer between 1 and the maximum number supported by XKCD, and set `number` to this new number.
Given our previous logic, the new comic will be fetched automatically.


## Bounds Checking
Make sure that the next button does not take you beyond the this.state.lastComic and you will have to modify prev() to not take you lower than 1.

Once you get all of these buttons working, celebrate, and turn in the URL to the lab.
