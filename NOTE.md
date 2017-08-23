# Developer's Note
This note is intended for developers. You might not need it

## Install locally
* `./install-linux.sh`

## Updating
* Update dependencies
    - `ncu -u`
    - `cd project-template`
    - `ncu -u`
    - `cd ..`
* Update version
    - `vim package.json`
    - Change the `version` key
    - `vim project-template/package.json`
    - Change the `dependencies/chimera-framework` version key

## Testing
* Testing Chimera:
    - `npm test`
* Testing Web Framework (__TODO:__ Create a better test)
    - `cd project-template`
    - `npm start`
    - Test manually

## Publish
* Commit and push git
    - `git add . -A`
    - `git commit -m 'commit description'`
    - `git push -u origin master`
* `sudo npm publish`