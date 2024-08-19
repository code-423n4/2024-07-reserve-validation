## Summary
The `Asset::isCollateral` function returns `false` at all times


## Details
The `Asset::isCollateral` function is supposed to check whether an asset is an instance of ICollateral or not. On the contrary, the function only returns a `false` on default without having to make any checks. This means irrespective of what the asset is, the `Asset::isCollateral` function will return `false` which can mislead the protocol.

See code snippet below
```javascript
/// @return If the asset is an instance of ICollateral or not
function isCollateral() external pure virtual returns (bool) {
@>    return false;
}

```
