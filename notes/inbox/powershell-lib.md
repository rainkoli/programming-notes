New-Item -ItemType Directory -Path ".\global-skills"
Get-ChildItem -Path ".\global-skills"

创建目录联接：前者为联接路径，后者为已存在的目标目录。

cmd /c mklink /J ".\global-skills\my-skills" ".\source-skills"

仅删除上面创建的目录联接，保留目标目录。

cmd /c rmdir ".\global-skills\my-skills"

Get-ChildItem -Path . -Directory -Recurse -Filter "node_modules" -Force | Remove-Item -Recurse -Force



netstat -ano | findstr 5346

tasklist | findstr 28920

Get-NetTCPConnection -LocalPort 5432



browser .chrome --headless --dump-dom https://www.google.com
