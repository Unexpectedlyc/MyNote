# JAVA8与Vue2中实现文件打包为zip并下载的demo

```java
//基于springboot实现
/**
 * 下载文件接口
 */
@Operation(summary = "下载文件")
@PreAuthorize("@ss.hasPermi('modelservice:ModelServiceModelInference:download')")
@RequiresPermissions("modelservice:ModelServiceModelInference:download")
@PostMapping("/downloadFileModelservice")
public ResponseEntity<?> downloadFileModelservice(@RequestParam(value = "filePath", required = true) String filePath) throws IOException {
    logger.info(filePath);
    Path zipFile = Paths.get(filePath).getParent().resolve("result_" + Paths.get(filePath).getFileName() + ".zip");
    
    try (ZipOutputStream zos = new ZipOutputStream(Files.newOutputStream(zipFile))) {
        Path rootPath = Paths.get(filePath);
        Files.walk(rootPath)
            .filter(path -> !Files.isDirectory(path))
            .forEach(path -> {
                ZipEntry zipEntry = new ZipEntry(rootPath.relativize(path).toString());
                try {
                    zos.putNextEntry(zipEntry);
                    Files.copy(path, zos);
                    zos.closeEntry();
                } catch (IOException e) {
                    logger.info("Failed to compress file: {} due to {}", path.toString(), e.getMessage());
                }
            });
    } catch (IOException e) {
        logger.info("Failed to create zip file due to {}", e.getMessage());
    }

    File file = new File(String.valueOf(zipFile));
    FileSystemResource resource = new FileSystemResource(file);

    // 检查文件是否存在
    if (!file.exists()) {
        return new ResponseEntity<>("File not found", HttpStatus.NOT_FOUND);
    }

    // 返回文件下载响应
    return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + file.getName() + "\"")
            .contentType(MediaType.APPLICATION_OCTET_STREAM)
            .body(resource);
}
```



```js
downloadFile() {
    const formData = new FormData();
    formData.append('filePath', this.form.result);

    const filePath = this.form.result.replace(/\\\\|\\/g, '/');
    const fileName = 'result_' + path.basename(filePath) + '.zip';

    downloadInferenceResult(formData)
        .then(response => {
            // 创建一个隐藏的 <a> 元素用于触发下载
            const url = window.URL.createObjectURL(new Blob([response]));
            const link = document.createElement('a');
            link.href = url;
            link.setAttribute('download', fileName); // 设置下载的文件名
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        })
        .catch(error => {
            console.error('下载失败:', error);
        });
}

```

