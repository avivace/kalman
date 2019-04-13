# Kalman 2D demo
### Kalman Filter implementation

### [Live demo](https://avivace.github.io/kalman)

Final project for the *Probabilistic models for decision making* course, from my MSc. An interactive 2D simulation of the Kalman Filter in use to reduce input noise

### Deploy

```bash
git clone
npm install
npm run serve
```

Deploy to `avivace.github.io/kalman`:

```bash
npm run deploy
```

> Development is done on the `develop` branch due to GitHub's restriction on branches for user pages (the build is deployed on the `master` branch and deployed to `avivace.github.io/kalman` from there).

### Stack

- [Vue.js](https://vuejs.org/) JS framework;
- [MuseUI](https://muse-ui.org) CSS framework;
- [Sylvester](http://sylvester.jcoglan.com/) Vector and matrix math;
- [CanvasRenderingContext2D web API](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D) to draw the simulation.

### References

- [An introduction to the Kalman Filter](http://www.cs.utexas.edu/~pstone/Courses/393Rfall13/readings/Welch+Bishop-TR-95.pdf)
- [Implementation of Kalman Filter with Python Language](https://arxiv.org/pdf/1204.0375.pdf)
- [Understanding the Basis of the Kalman Filter via a Simple and Intuitive Derivation](https://courses.engr.illinois.edu/ece420/sp2017/UnderstandingKalmanFilter.pdf)