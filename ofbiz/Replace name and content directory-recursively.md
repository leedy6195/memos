```PowerShell

Get-ChildItem -Recurse | ForEach-Object {
    if (-not $_.PSIsContainer) {
        # 파일 내의 텍스트 대체 (case-sensitive)
        (Get-Content $_.FullName -Raw) | ForEach-Object {
            $_ -creplace "example", "beacon"
        } | Set-Content $_.FullName
    }

    # 파일 이름 변경 (case-sensitive)
    $newName = $_.Name -creplace "example", "beacon"
    if ($_ -ne $newName) {
        Rename-Item -Path $_.FullName -NewName $newName -ErrorAction SilentlyContinue
    }
}

```