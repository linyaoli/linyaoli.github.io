---
layout: post
title: mocha test exceeds timeout
tags: programming
---
For the following case, you may see macha test exceeds timeout even with `timeout(6000)` setup.
Be sure to resolve promise at the end setTimeout.
~~~
it('blah', () => ) {
  this.timeout(6000)
  return new Promise((resolve) => {
    setTimeout(() => {
      helper.request().get('/testurl/' + id)
      .then()//whatever
    resolve() <=== don't forget this!
    }, 5000)
  })
}
~~~
