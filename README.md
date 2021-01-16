### Promise

```tsx
// 定义 Promise 的三种状态
const Pending = 'pending';
const Resolved = 'resolved';
const Rejected = 'rejected';

type ExecuteType = (resolve: (value?: unknown) => void, reject: (reason: any) => void) => void;

class Promise {
  status: string;
  value: unknown;
  reason: any;
  onFulfilleds: Array<Function>;
  onRejecteds: Array<Function>;
  constructor(execute: ExecuteType) {
    this.status = Pending; // 记录初始状态
    this.value = undefined; // 记录执行成功的值
    this.reason = undefined; // 记录执行失败的值
    this.onFulfilleds = []; // 记录执行成功回调函数
    this.onRejecteds = []; // 记录执行失败回调函数

    // Promise 的 resolve 方法
    const resolve = (value?: unknown): void => {
      if (this.status === Pending) {
        this.value = value;
        this.status = Resolved;
        this.onFulfilleds.forEach((fn) => fn());
      }
    };

    // Promise 的 reject 方法
    const reject = (reason?: any): void => {
      if (this.status === Pending) {
        this.reason = reason;
        this.status = Rejected;
        this.onRejecteds.forEach((fn) => fn());
      }
    };
    execute(resolve, reject);
  }

  // then 方法及链式调用
  then(
    onfulfilled?: ((value: unknown) => unknown) | null | undefined,
    onrejected?: ((reason: any) => any) | null | undefined,
  ) {
    return new Promise((resolve, reject) => {
      // onfulfilled
      if (typeof onfulfilled === 'function') {
        // setTimeout 模拟微任务
        this.status === Resolved &&
          setTimeout(() => {
            resolve(onfulfilled(this.value));
          });

        this.status === Pending &&
          this.onFulfilleds.push(() =>
            setTimeout(() => {
              resolve(onfulfilled(this.value));
            }),
          );
      }

      // onrejected
      if (typeof onrejected === 'function') {
        this.status === Rejected &&
          setTimeout(() => {
            reject(onrejected(this.reason));
          });
        this.status === Pending &&
          this.onRejecteds.push(() =>
            setTimeout(() => {
              reject(onrejected(this.reason));
            }),
          );
      }
    });
  }

  // Promise.catch 方法 是then(null,onrejected) 的实现
  catch = (onrejected: (reason: any) => void) => {
    if (typeof onrejected === 'function') this.then(null, onrejected);
  };

  // resolve 静态方法: TODO: 未实现
  static resolve(value: unknown) {
    return new Promise((resolve) => {
      resolve(value);
    });
  }

  // TODO: 待完成
  static race() {}

  // Promise.all 的实现
  static all(PromiseList: Array<Promise>) {
    try {
      // TODO: 原生Promise 的js语法树不匹配报错实现？
      return new Promise((resolve, reject) => {
        let flag: boolean = false; // 记录传入的Promise是否有失败的
        const newList: Array<unknown> = [];
        PromiseList.forEach((promise) => {
          promise.then(
            (res) => {
              newList.push(res);
            },
            (err) => {
              if (err) flag = true;
              reject(err);
            },
          );
        });
        // 在promise.then之后执行 TODO: PromiseList的循环不需要结束,flag能否去掉
        setTimeout(() => {
          if (!flag) {
            resolve(newList);
          }
        });
      });
    } catch (err) {
      console.log(err);
      return new Promise(() => {});
    }
  }

  // Promise.allSettled 的实现 TODO: 待完成
  static allSettled() {}
}

export default Promise;

```



