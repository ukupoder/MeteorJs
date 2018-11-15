# Installimine

## OSX/Linux
`curl https://install.meteor.com/ | sh`



## Windows
```
choco install meteor

@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"

choco upgrade chocolatey
```

# Rakenduse tegemine

`meteor create simple-todos`

# Rakenduse käivitamine

`meteor`


