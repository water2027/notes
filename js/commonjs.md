```javascript
function require(modulePath) {
	var moduleId = getModuleId(modulePath)
	if (cache[moduleId]) {
		return cache[moduleId]
	}
	
	function _require(exports, require, module, __filename, __dirname) {
		// 执行模块代码
	}
	
	var module = {
		// ...
		exports: {},
	}
	var exports = module.exports
	var __filename = moduleId
	var __dirname = getDirname(__filename)
	
	_require.call(exports, exports, require, module, __filename, __dirname)
	cache[moduleId] = module.exports
	
	return module.exports
}
```

