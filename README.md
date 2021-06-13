## Pure Process

Primitives for business logic structuring.

#### Install

```
npm install pure-process
```

#### Promise chain exits

<details>
<summary>JavaScript</summary>

```javascript
const {exit} = require('pure-process');

const EXIT_CODE = {
    needLogin: 'needLogin'
};

const getUser = async() => {
    const {user, latency} = await fetchUser();
    if (!user) {
        exit(EXIT_CODE.needLogin, {latency});
    }

    return user;
};

const getPosts = async(user) => ({
    posts: await fetchPosts(user.id)
});

const process = () => Promise.resolve()
    .then(getUser)
    .then(getPosts)
    .then(...exit());

const output = await process();

if (output.exitCode === EXIT_CODE.needLogin) {
    output.latency;
} else {
    output.exitCode; // main (default)
    output.posts;
}
```
</details>

<details>
<summary>TypeScript</summary>

```typescript
const {createExit} = require('pure-process');

enum ExitCode {
    NeedLogin = 'NEED_LOGIN',
    NoPosts = 'NO_POSTS'
};

const exit = createExit<{
    exitCode: ExitCode.NeedLogin;
    latency: number;
} | {
    exitCode: ExitCode.NoPosts
}>();

const getUser = async() => {
    const {user, latency} = await fetchUser();
    if (!user) {
        exit(ExitCode.NeedLogin, {latency});
    }

    return user;
};

const getPosts = async(user: User) => ({
    posts: await fetchPosts(user.id)
});

const process = () => Promise.resolve()
    .then(getUser)
    .then(getPosts)
    .then(...exit());

const output = await process();

if (output.exitCode === ExitCode.NeedLogin) {
    output.latency;
} else {
    output.exitCode; // main (default)
    output.posts;
}
```
</details>

#### Sequential and parallel execution

```javascript
const {pipeP, parallelMerge} = require('pure-process');

const stepA = () => ({
    resultA: 1
});

const stepB = ({resultA}) => ({
    resultB: resultA + 1
});

const stepC = ({resultA}) => ({
    resultC: resultA + 2
});

const process = () => Promise.resolve()
    .then(stepA)
    .then((data) => Promise.all([
            stepB(data),
            stepC(data)
        ]).then(([result1, result2]) => ({
            ...data,
            ...result1,
            ...result2
        }))
    );

// Equivalent
const process = pipeP(
    stepA,
    parallelMerge(
        stepB,
        stepC
    )
);

expect(await process()).toEqual({
    resultA: 1,
    resultB: 2,
    resultC: 3
});
```

#### Sequences with exits

```javascript
const process = pipeP(
    stepA,
    stepB,
    exit.pipe
);
```

#### Skip errors

```javascript
const {skipErrors} = require('pure-process');

const stepA = async(data) => ({
    ...data,
    resultA: await fetchCritical()
});

const stepB = async(data) => ({
    ...data,
    resultB: await fetchNonCritical()
});

const process = (data) => Promise.resolve(data)
    .then(stepA)
    .then(skipErrors(stepB));

await process({
    // Optional
    logError: console.log
});
```