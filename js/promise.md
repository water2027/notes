```javascript
const FULFILLED = 'fulfilled'
const PENDING = 'pending'
const REJECTED = 'rejected'

function runMicroTask(fn) {
  if (typeof queueMicrotask === 'function') {
    queueMicrotask(fn)
  }
  else if (
    typeof process === 'object'
    && typeof process?.nextTick === 'function'
  ) {
    process.nextTick(fn)
  }
  else if (typeof MutationObserver === 'function') {
    const text = document.createTextNode('')
    const observer = new MutationObserver(fn)
    observer.observe(text, {
      characterData: true,
    })
    text.data = '1'
  }
  else {
    setTimeout(fn)
  }
}

function isPromiseLike(obj) {
  return typeof obj?.then === 'function'
}

function isIterable(obj) {
  return (
    obj !== null
    && obj !== undefined
    && typeof obj[Symbol.iterator] === 'function'
  )
}

class MyPromise {
  #state
  #value
  #handlers = []
  constructor(executor) {
    // 初始为pending状态
    this.#state = PENDING
    this.#handlers = []
    const resolve = (val) => {
      this.#resolvePromise(val)
    }
    const reject = (val) => {
      this.#setState(REJECTED, val)
    }
    try {
      executor(resolve, reject)
    }
    catch (error) {
      reject(error)
    }
  }

  #resolvePromise(x) {
    if (x === this) {
      this.#setState(REJECTED, new Error('TypeError'))
      return
    }
    if (x instanceof Promise) {
      x.then(
        (res) => {
          this.#setState(FULFILLED, res)
        },
        (err) => {
          this.#setState(REJECTED, err)
        }
      )
      return
    }
    if (isPromiseLike(x)) {
      let hasCalled = false
      try {
        x.then(
          (y) => {
            if (hasCalled)
              return
            this.#resolvePromise(y)
            hasCalled = true
          },
          (r) => {
            if (hasCalled)
              return
            this.#setState(REJECTED, r)
            hasCalled = true
          }
        )
      }
      catch (error) {
        if (hasCalled)
          return
        this.#setState(REJECTED, error)
      }
      return
    }
    this.#setState(FULFILLED, x)
  }

  #setState(state, val) {
    // a+规范中要求fulfilled和reject状态不可改变
    if (this.#state !== PENDING)
      return
    this.#state = state
    this.#value = val
    this.#runTask()
  }

  #runTask() {
    runMicroTask(() => {
      for (const handler of this.#handlers) {
        handler()
      }
      this.#handlers = []
    })
  }

  then(onFulfilled, onReject) {
    return new MyPromise((resolve, reject) => {
      this.#handlers.push(() => {
        const callback
					= this.#state === FULFILLED ? onFulfilled : onReject
        try {
          if (typeof callback === 'function') {
            const res = callback(this.#value)
            resolve(res)
          }
          else {
            this.#state === FULFILLED
              ? resolve(this.#value)
              : reject(this.#value)
          }
        }
        catch (error) {
          reject(error)
        }
      })
      // 如果已经完成了, 就立刻执行
      if (this.#state !== PENDING) {
        this.#runTask()
      }
    })
  }

  catch(onReject) {
    return this.then(undefined, onReject)
  }

  finally(onFinally) {
    return this.then(
      (res) => {
        onFinally()
        return res
      },
      (err) => {
        onFinally()
        throw err
      }
    )
  }

  static resolve(x) {
    if (x instanceof MyPromise) {
      return x
    }
    if (isPromiseLike(x)) {
      return new MyPromise((resolve, reject) => {
        x.then(resolve, reject)
      })
    }
    return new MyPromise((resolve, reject) => {
      resolve(x)
    })
  }

  static reject(x) {
    return new MyPromise((resolve, reject) => {
      reject(x)
    })
  }

  static try(fn, ...args) {
    return new MyPromise((resolve, reject) => {
      try {
        resolve(fn(...args))
      }
      catch (err) {
        reject(err)
      }
    })
  }

  static all(promises) {
    if (!isIterable(promises)) {
      throw new Error('TypeError')
    }
    return new MyPromise((resolve, reject) => {
      const arr = Array.from(promises)
      if (arr.length === 0) {
        resolve([])
      }
      const result = Array.from({ length: arr.length })
      let i = 0
      arr.forEach((p, index) => {
        MyPromise.resolve(p).then((res) => {
          result[index] = res
          i++
          if (i === arr.length) {
            resolve(result)
          }
        }, reject)
      })
    })
  }

  static allSettled(promises) {
    if (!isIterable(promises)) {
      throw new Error('TypeError')
    }
    return new MyPromise((resolve, reject) => {
      const arr = Array.from(promises)
      if (arr.length === 0) {
        resolve([])
      }
      const result = Array.from({ length: arr.length })
      let i = 0
      arr.forEach((p, index) => {
        MyPromise.resolve(p)
          .then(
            (res) => {
              result[index] = {
                value: res,
                status: FULFILLED,
              }
            },
            (err) => {
              result[index] = {
                reason: err,
                status: REJECTED,
              }
            }
          )
          .finally(() => {
            i++
            if (i === arr.length) {
              resolve(result)
            }
          })
      })
    })
  }

  static any(promises) {
    if (!isIterable(promises)) {
      throw new Error('TypeError')
    }
    return new MyPromise((resolve, reject) => {
      const arr = Array.from(promises)
      if (arr.length === 0) {
        reject([])
      }
      const result = Array.from({ length: arr.length })
      let i = 0
      arr.forEach((p, index) => {
        MyPromise.resolve(p).then(resolve, (err) => {
          i++
          result[index] = err
          if (i === arr.length) {
            reject(result)
          }
        })
      })
    })
  }

  static race(promises) {
    if (!isIterable(promises)) {
      throw new Error('TypeError')
    }
    return new MyPromise((resolve, reject) => {
      for (const p of promises) {
        MyPromise.resolve(p).then(resolve, reject)
      }
    })
  }

  static withResolvers() {
    let resolve, reject
    const promise = new MyPromise((res, rej) => {
      resolve = res
      reject = rej
    })
    return {
      promise,
      resolve,
      reject,
    }
  }
}
```