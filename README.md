# README

## Demo

- Setup Rails 5.1 with Redux
- Extend API with new endpoint
- Add new redux elements
- Further readings

## Setup

``` bash
rvm use 2.4.1@ToDoRails51ReactV1
git init
gem install bundler
gem install rails -v='5.1.2'
rails new todos --webpack=react
cd todos
yarn add --dev jest enzyme babel-preset-stage-2 react-addons-test-utils redux-mock-store
echo "{\n  \"presets\": [\"es2015\", \"react\", \"stage-2\"]\n}\n" > .babelrc
yarn add react-redux redux-undo redux
echo "rails: bin/rails s\nwebpack: ./bin/webpack-dev-server" > Procfile
```

In `config/environments/development.rb` add webpack dev server config:

```
config.x.webpacker[:dev_server_host] = "http://localhost:8080"
```

Add `gem 'forman'` to the `:development` group in the `Gemfile` and

``` bash
bundle
```

In `package.json` add the test script and configuration

``` json
  "scripts": {
    "test": "NODE_PATH='./node_modules:../app/javascript' jest --watch --coverage"
  },
  "jest": {
    "roots": [
      "app/javascript"
    ]
  },
```

Add `/coverage` to `.gitignore`

You can now start the application with

``` bash
foreman start
```

# TODO

- [ ] Finish initial app set up
- [ ] Add a ToDo model
- [ ] Add a ToDo controller
- [ ] Update client to load and save data
- [ ] Add more links
- [ ] Figure out why there was no .babelrc or webpack scripts in bin/

# Useful Links:

## Rails 5.1:

## Redux:

[A cartoon intro to redux](https://code-cartoons.com/a-cartoon-intro-to-redux-3afb775501a6)
[An introduction to redux](https://www.smashingmagazine.com/2016/06/an-introduction-to-redux/)
