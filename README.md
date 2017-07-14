# README

``` bash
rvm use 2.4.1@ToDoRails51ReactV1
git init
gem install bundler
gem install rails -v='5.1.2'
rails new todos --webpack=react
cd todos
yarn add --dev jest babel-preset-stage-2 react-addons-test-utils redux-mock-store
echo "{\n  \"presets\": [\"es2015\", \"react\", \"stage-2\"]\n}\n" > .babelrc
```

Modify babel.rc