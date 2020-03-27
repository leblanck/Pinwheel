# Pinwheel


Blog built using [Hugo](https://gohugo.io/) and hosted with [Firebase](https://firebase.google.com/)

Please see my [blog post](https://pinwheel.dev/blog/building-pinwheel-with-hugo-and-firebase/) on building this site for more information on the build process. 

**Install Prerequisites**
Hugo & firebase are both needed to run this locally and to deploy to a firebase project. You shold already have a firebase account/project setup.

`brew cask install hugo && brew cask install firebase`

**Setup**
1. `git clone https://github.com/leblanck/Pinwheel.git`
2. `cd pinwheel`
3. Run `firebase login` to authenticate to your firebase account
4. Run `hugo server` (spins up local hugo site at `localhost:1313`)
5. Edit content/make changes. These will be visible in realtime at localhost:1313
5. Run `hugo && firebase deploy --project firebaseProjectID` to build/compile your hugo site and deploy to firebase
