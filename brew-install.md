# homebrew install with tap

### Install XCode
You'll need to install [Xcode](https://developer.apple.com/xcode/downloads/) and the ```Commandline Tools``` first.  This installation was successful with Xcode 8.

```shell
xcode-select --install
softwareupdate --install --all 
```

### Install Homebrew
```shell
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Add homebrew tap
```shell
brew tap caskroom/cask
brew tap caskroom/drivers
brew tap caskroom/versions
brew tap caskroom/eid
brew tap caskroom/fonts
brew tap homebrew/python
brew tap homebrew/boneyard
brew tap homebrew/bundle
brew tap homebrew/completions
brew tap homebrew/core
brew tap homebrew/dupes
brew tap homebrew/fuse
brew tap homebrew/x11
```

brew tap metacollin/gnuradio
brew tap chleggett/gqrx
brew tap chleggett/gr-osmosdr
brew tap dskecse/tap
