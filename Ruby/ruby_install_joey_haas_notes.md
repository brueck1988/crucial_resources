Ruby Setup
run brew update
then brew install rbenv
run rbenv init and copy form the output something that looks like eval "$(rbenv init -)"
load into your terminal using this command code .zshrc this should open a file
then copy and paste into page
eval "$(rbenv init -)"
alias bes="bundle exec rspec"
run rbenv install 2.5.3
run rbenv local 2.5.3
run rbenv global 2.5.3
Rails Setup
type in command line gem install rails --version 5.2.5