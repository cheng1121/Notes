# node.js 报错

## 1. error:0308010C:digital envelope routines::unsupported
出现这个错误是因为 node.js V17版本中最近发布的OpenSSL3.0, 而OpenSSL3.0对允许算法和密钥大小增加了严格的限制，

在node.js V17以前一些可以正常运行的的应用程序,但是在 V17 版本可能会抛出以下异常:



```ruby
node:internal/crypto/hash:67
  this[kHandle] = new _Hash(algorithm, xofLen);
                  ^

Error: error:0308010C:digital envelope routines::unsupported
    at new Hash (node:internal/crypto/hash:67:19)
    at Object.createHash (node:crypto:130:10)
    at module.exports.__webpack_modules__.57442.module.exports (/Users/workspace/React/umiapp/node_modules/@umijs/deps/compiled/webpack/4/bundle4.js:135907:62)
    at NormalModule._initBuildHash (/Users/workspace/React/umiapp/node_modules/@umijs/deps/compiled/webpack/4/bundle4.js:109317:16)
    at handleParseError (/Users/workspace/React/umiapp/node_modules/@umijs/deps/compiled/webpack/4/bundle4.js:109371:10)
    at /Users/workspace/React/umiapp/node_modules/@umijs/deps/compiled/webpack/4/bundle4.js:109403:5
    at /Users/workspace/React/umiapp/node_modules/@umijs/deps/compiled/webpack/4/bundle4.js:109258:12
    at /Users/workspace/React/umiapp/node_modules/@umijs/deps/compiled/webpack/4/bundle4.js:61157:3
    at iterateNormalLoaders (/Users/workspace/React/umiapp/node_modules/@umijs/deps/compiled/webpack/4/bundle4.js:60998:10)
    at Array.<anonymous> (/Users/workspace/React/umiapp/node_modules/@umijs/deps/compiled/webpack/4/bundle4.js:60989:4) {
  opensslErrorStack: [ 'error:03000086:digital envelope routines::initialization error' ],
  library: 'digital envelope routines',
  reason: 'unsupported',
  code: 'ERR_OSSL_EVP_UNSUPPORTED'
}

Node.js v17.0.1
✨  Done in 1.92s.
```

目前可以通过运行以下命令行临时解决这个问题



```bash
export NODE_OPTIONS=--openssl-legacy-provider
```

