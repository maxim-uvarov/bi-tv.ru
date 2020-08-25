---
title: "BI-TV #23: Редактирование кода скриптов Power Query в VS Code - решение Ben Gribaudo:"
date: 2020-06-13
header:
//  og_image: /assets/images/_22.png
categories:
  - Video
tags:
  - Power BI
---
Редактирование кода скриптов Power Query в VS Code - решение Ben Gribaudo:
[https://bengribaudo.com/blog/2020/07/16/5356/editing-report-spreadsheet-mashups-in-vscode#more-5356]

3. Устанавливаем VS Code [https://code.visualstudio.com/Download]
4. В VS Code устанавливаем расширение "Power Query / M Language" (ctrl+shift+P - установить расширение - вводим название)
1. Устанавливаем PowerShell Core 7 c github: [https://github.com/powershell/powershell#get-powershell]
2. Чтобы заработала возможность установки скриптов из галереи используем команду 
```
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
```
Решение найдено [здесь](https://www.myerrorsandmysolutions.com/unable-to-resolve-package-source-https-www-powershellgallery-com-api-v2/)
5. Устанавливаем Data Mashup Cmdlet, для этого вводим PowerShell Core 7 команду: 
```Install-Module -Name DataMashup -AllowPrerelease'```
6. В скрипте функции заменяем code.exe на путь до VS Code на вашем компьютере:
```
function Edit-DataMashup {
    param (
        [Parameter(Mandatory=$True, Position=0)]
        [string]$File
    )
   
    $originalMashup = Export-DataMashup -Raw $File -ErrorAction Stop
       
    $tempFile = New-TemporaryFile -ErrorAction Stop
       
    try {
        $tempFile = Rename-Item $tempFile ($tempFile.Name + ".m") -PassThru -ErrorAction Stop
   
        $originalMashup | Out-File $tempFile -NoNewline -ErrorAction Stop
        Start-Process "code.exe" "`"$($tempFile)`"" -Wait 
        $postEditMashup = Get-Content $tempFile -Raw -ErrorAction Stop
           
        if ($originalMashup -eq $postEditMashup) { 
            $successful = $true
            return
        }
           
        Import-DataMashup -Raw $File -Experimental -Mashup $postEditMashup -ErrorAction Stop
        $successful = $true
    }   
    finally {
        if ($successful -ne $true) {
            Write-Error "Failed to Save Modified Power Query mashup:`r`n$($postEditMashup)"
        }
      
        Remove-Item $tempFile
    }
}
```
7. Редактируем код Power Query XLSX и PBI файлов в VS Code командой:
`Edit-DataMashup SomeFile.xlsx`



/*
<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='https://www.youtube.com/embed/ShOfVylvAbM' frameborder='0' allowfullscreen></iframe></div>
*/