## etcdv3
解决`etcdv3`不能依赖`grpc@v1.27.0`版本问题.

本来解决方法最简单的是直接替换`v1.26.0`版本即可。但是我的另外一个项目会用`drand`库，这个库又会用`grpc@v1.27.0`。这会导致库的冲突。

没办法只能原生来解决。

可以看这里如何解决的。[commit merged](https://github.com/grpc/grpc-go/pull/3309/commits/15ec624bc28b97ac9aaedf4b2241df391be82a57#diff-c6a2b1bb9d7feeb56f43abf7caadeed1)

- 复制`grpc@v1.27.0`的代码。
- 修改`balancer/balancer.go`文件内容。实际就是一个重命名`type`即可。
```go
// PickOptions is a type alias of PickInfo for legacy reasons.
//
// Deprecated: use PickInfo instead.
type PickOptions = PickInfo

// PickInfo contains additional information for the Pick operation.
type PickInfo struct {
	// FullMethodName is the method name that NewClientStream() is called
```
- 修改`resolver/resolver.go`里面的内容：
```go
Metadata interface{}
}

// BuildOption is a type alias of BuildOptions for legacy reasons.
//
// Deprecated: use BuildOptions instead.
type BuildOption = BuildOptions

// BuildOptions includes additional information for the builder to create
// the resolver.
type BuildOptions struct {
@@ -235,11 +230,6 @@ type Builder interface {
	Scheme() string
}

// ResolveNowOption is a type alias of ResolveNowOptions for legacy reasons.
//
// Deprecated: use ResolveNowOptions instead.
type ResolveNowOption = ResolveNowOptions

// ResolveNowOptions includes additional information for ResolveNow.
type ResolveNowOptions struct{}
```

- 修改`go.mod`即可：
```go
replace google.golang.org/grpc => ./grpc@v1.27.0
```
