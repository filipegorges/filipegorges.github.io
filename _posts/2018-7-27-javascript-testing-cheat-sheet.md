The following is a series of testing frameworks and libraries suggestions for JS/React projects, basic syntax and usage, as well as pointers to official sites and documentation.

The suggestions are based on the following Udemy course: https://www.udemy.com/js-com-tdd-na-pratica/ (in portuguese) created by [Willian Justen](https://github.com/willianjusten).

NPM packages:

```
npm i --save-dev mocha chai sinon sinon-chai enzyme chai-enzyme node-fetch jsdom jsdom-global nyc coveralls
```

Quick rundown of each package:

Package | Usefulness
--- | ---
Mocha | Test runner, provides `describe`, `it`, `context`, `beforeEach`, etc.
Chai | Assertion library, provides `expect`, `to`, `have`, `be`, `a`, `an`, etc.
Sinon | Test spies, stubs and mocks, provides `sinon.stub`, etc.
SinonChai | Integration between Sinon and Chai, provides `calledWith`, `calledOn`, etc.
Enzyme | Assertion library for testing React components.
ChaiEnzyme | Integration between Chai and Enzyme, provides `shallow`, `mount`, `render`, etc.
Node-Fetch | Brings `window.fetch` to Node.js, allowing to test fetch calls, etc.
jsdom/global | Provides access to `document` and `window` elements on Node.js for testing.
NYC | Test coverage report
Coveralls | Code coverage insight

### Mocha
[Official site](https://mochajs.org/) and [Documentation](https://github.com/mochajs/mocha/wiki)

Syntax example:
```
describe('componentName', () => {
  context('givenContext', () => {
    it('should meet this expectation...', () => {
      ...
    });
  });
});
```

Usage example:
```
describe('<FullHeader />', () => {
   it('should have header tag when mount', () => {
        const wrapper = shallow(<FullHeader />);
        expect(wrapper.find('header')).to.have.length(1);
   });

   context('title', () => {
       it('should have h1 tag when title passed', () => {
           const wrapper = shallow(<FullHeader title="TDD"/>);
           expect(wrapper.find('h1')).to.have.length(1);
       });
   });
});
```

### Chai
[Official site](http://www.chaijs.com/) and [Documentation](http://www.chaijs.com/api/bdd/)

Syntax example:
```
import chai, { expect } from 'chai';

expect(object).to.exist;
expect(object).to.have.length(5);
expect(object).to.be.equal('foo');
```

Usage example:
```
import { expect } from 'chai';
import converToHumanTime from '../src/ConvertToHumanTime';

describe('ConvertToHumanTime', () => {
  // 0ms === 0:00
  it('should receive 0ms and convert to 0:00', () => {
    expect(converToHumanTime(0)).to.be.equal('0:00');
  });

  // 1000ms === 0:01
  it('should receive 1000ms and convert to 0:01', () => {
    expect(converToHumanTime(1000)).to.be.equal('0:01');
  });

  // 11000 === 0:11
  it('should receive 11000 and convert to 0:11', () => {
    expect(converToHumanTime(11000)).to.be.equal('0:11');
  });

  // 60000 === 1:00
  it('should receive 60000 and convert to 1:00', () => {
    expect(converToHumanTime(60000)).to.be.equal('1:00');
  });
});
```

### Sinon
[Official site](http://sinonjs.org/) and [Documentation](http://sinonjs.org/releases/v6.1.4/)

Syntax example:
```
import sinon from 'sinon';

let consoleStub;
// before usage
consoleStub = sinon.stub(console, 'info');
// after usage, clearing modified state
consoleStub.restore();
```

Usage example:
```
import sinon from 'sinon';
global.fetch = require('node-fetch');

let stubbedFetch;
let promise;

beforeEach(() => {
  stubbedFetch = sinon.stub(global, 'fetch');
  promise = stubbedFetch.resolves({ json: () => ({ album: 'name' }) });
});

afterEach(() => {
  stubbedFetch.restore();
});
```

### Sinon Chai
[Official site](https://github.com/domenic/sinon-chai) and [Documentation](https://github.com/domenic/sinon-chai#assertions)

Syntax example:
```
import chai, { expect } from 'chai';
import sinon from 'sinon';
import sinonChai from 'sinon-chai';

chai.use(sinonChai)

expect(mySpy).to.have.been.calledWith("foo");
expect(mySpy).to.have.been.calledOnce;
expect(mySpy).to.have.been.called;
```

Usage example:
```
import chai, { expect } from 'chai';
import sinon from 'sinon';
import sinonChai from 'sinon-chai';

chai.use(sinonChai)

global.fetch = require('node-fetch');

describe('Album', () => {
  let spotify;
  let stubbedFetch;
  let promise;

  beforeEach(() => {
    spotify = new SpotifyWrapper({
      token: 'foo',
    });

    stubbedFetch = sinon.stub(global, 'fetch');
    promise = stubbedFetch.resolves({ json: () => ({ album: 'name' }) });
  });

  afterEach(() => {
    stubbedFetch.restore();
  });
  
  describe('getTracks', () => {

    it('should call fetch method', () => {
      const albumTrack = spotify.album.getTracks();
      expect(stubbedFetch).to.have.been.calledOnce;
    });

    it('should call the correct URL', () => {
      const albumTrack = spotify.album.getTracks('6akEvsycLGftJxYudPjmqK')
      expect(stubbedFetch).to.have.been.calledWith('https://api.spotify.com/v1/albums/6akEvsycLGftJxYudPjmqK/tracks')
    });

    it('should return the correct data frin Promise', () => {
      const tracks = spotify.album.getTracks('6akEvsycLGftJxYudPjmqK');
      tracks.then((data) => {
        expect(data).to.be.eql({ album: 'name' });
      });
    });
  });
```

### Enzyme
[Official site](http://airbnb.io/enzyme/) and [Documentation](https://airbnb.io/enzyme/docs/api/)

Syntax example:
```
import React from 'react';
import { shallow } from 'enzyme';

const wrapper = shallow(<MyComponent />);
const wrapper = shallow((
    <MyComponent>
        <div className="unique" />
    </MyComponent>
));
```

Usage example:
```
import React from 'react';
import { expect } from 'chai';
import { render } from 'enzyme';

import Foo from './Foo';

describe('<Foo />', () => {
  it('renders three `.foo-bar`s', () => {
    const wrapper = render(<Foo />);
    expect(wrapper.find('.foo-bar')).to.have.lengthOf(3);
  });
});
```

### Enzyme Chai
[Official site](https://github.com/producthunt/chai-enzyme) and [Documentation](https://github.com/producthunt/chai-enzyme#assertions)

Syntax example:
```
const wrapper = mount(<Fixture />); // mount/render/shallow when applicable

expect(wrapper.find('#checked')).to.be.checked();

expect(wrapper.find('span')).to.have.className('child');

expect(wrapper).to.contain(<User index={1} />);
expect(wrapper).to.contain([<User index={2} />, <User index={3} />]);


expect(wrapper).to.containMatchingElement(<User name='John' />);
```

Usage example:
```
import React from 'react';
import chai, { expect } from 'chai';
import chaiEnzyme from 'chai-enzyme';
import { shallow } from 'enzyme';
import FullHeader from '../../src/Main';

chai.use(chaiEnzyme());

describe('<FullHeader />', () => {
   it('should have header tag when mount', () => {
        const wrapper = shallow(<FullHeader />);
        expect(wrapper.find('header')).to.have.length(1);
   });

   context('title', () => {
       it('should have h1 tag when title passed', () => {
           const wrapper = shallow(<FullHeader title="TDD"/>);
           expect(wrapper.find('h1')).to.have.length(1);
       });

       it('should not have h1 tag when title is not passed', () => {
           const wrapper = shallow(<FullHeader />);
           expect(wrapper.find('h1')).to.have.length(0);
       });

       it('should have h1 tag with the title passed', () => {
           const wrapper = shallow(<FullHeader title="TDD" />);
           expect(wrapper.find('h1').props().children).to.be.equal("TDD");
       });
   });
```

### Node Fetch
[Official site](https://github.com/bitinn/node-fetch) and [Documentation](https://github.com/bitinn/node-fetch#api)

Syntax example:
```
global.fetch = require('node-fetch');

// plain text or html
fetch('https://github.com/')
	.then(res => res.text())
	.then(body => console.log(body));

// json
fetch('https://api.github.com/users/github')
	.then(res => res.json())
	.then(json => console.log(json));

```

Usage example:
```
import chai, { expect } from 'chai';
import sinon from 'sinon';
import sinonChai from 'sinon-chai';

chai.use(sinonChai);

global.fetch = require('node-fetch');

let spotify;
  let stubbedFetch;

  beforeEach(() => {
    spotify = new SpotifyWrapper({
      token: 'foo'
    });
    stubbedFetch = sinon.stub(global, 'fetch');
    stubbedFetch.resolves({ json: () => {} });
  });

  afterEach(() => {
    stubbedFetch.restore();
  });

describe('spotify.search.artists', () => {
    it('should call fetch function', () => {
      const artists = spotify.search.artists('Incubus');
      expect(stubbedFetch).to.have.been.calledOnce;
    });

    it('should fetch with the correct URL', () => {
      const artists = spotify.search.artists('Incubus');
      expect(stubbedFetch).to.have.been.calledWith('https://api.spotify.com/v1/search?q=Incubus&type=artist');

      const artists2 = spotify.search.artists('Muse');
      expect(stubbedFetch).to.have.been.calledWith('https://api.spotify.com/v1/search?q=Muse&type=artist');
    });
});
```

### JSDOM/ jsdom-global
[Official site](https://github.com/jsdom/jsdom) and [Documentation](https://github.com/jsdom/jsdom#customizing-jsdom)

Syntax example:
```
const jsdom = require("jsdom");
const { JSDOM } = jsdom;

const dom = new JSDOM(`<!DOCTYPE html><p>Hello world</p>`);
console.log(dom.window.document.querySelector("p").textContent); // "Hello world"
```

Usage example:
```
import 'jsdom-global/register';
import chai, { expect } from 'chai';

 it('should create an instance of SpotifyWrapper', () => {
    const spotify = new SpotifyWrapper({});
    expect(spotify).to.be.an.instanceof(SpotifyWrapper);
  });
  
  it('should receive apiURL as an option', () => {
    const spotify = new SpotifyWrapper({
      apiURL: 'blabla',
    });

    expect(spotify.apiURL).to.be.equal('blabla');
  });
  
   const markup = `
    <img class="album-image" src="https://i.scdn.co/image/59a536f0bf0ddaa427e4c732a061c33fe7578757" alt="The Essential Incubus">
    <p class="album-title">The Essential Incubus</p>
    <p class="album-artist">Incubus</p>
    <p class="album-counter">18 Músicas</p>
  `;

  it('should create and append the markup given a correct data', () => {
    const element = document.createElement('div');
    renderAlbumInfo(data, element);

    expect(element.innerHTML).to.be.eql(markup);
  });
```

### NYC and Coveralls
NYC [Official site](https://istanbul.js.org/) and [Documentation](https://istanbul.js.org/docs/tutorials/mocha/)

Coveralls [Official site](https://coveralls.io/) and [Documentation](https://docs.coveralls.io/api-introduction)

Output example:
```
-----------|----------|----------|----------|----------|-------------------|
File       |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
-----------|----------|----------|----------|----------|-------------------|
All files  |      100 |      100 |      100 |      100 |                   |
 album.js  |      100 |      100 |      100 |      100 |                   |
 config.js |      100 |      100 |      100 |      100 |                   |
 index.js  |      100 |      100 |      100 |      100 |                   |
 search.js |      100 |      100 |      100 |      100 |                   |
 utils.js  |      100 |      100 |      100 |      100 |                   |
-----------|----------|----------|----------|----------|-------------------|
```

Usage example:
```
## package.json
{
  ...
  "scripts": {
    ...
    "test": "./node_modules/.bin/mocha tests/**/*.spec.js",
    "test:tdd": "./node_modules/.bin/mocha tests/**/*.spec.js --watch",
    "test:coverage": "nyc npm test",
    "coveralls": "npm run test:coverage && nyc report --reporter=text-lcov | coveralls"
  },
  "files": [
    "dist",
    "lib"
  ],
  "nyc": {
    "reporter": [
      "text",
      "html"
    ],
    "exclude": [
      "tests/**"
    ]
  },
  ...
}

## terminal
# for basic coverage
npm run test:coverage

# for coveralls submission
npm run coveralls
```