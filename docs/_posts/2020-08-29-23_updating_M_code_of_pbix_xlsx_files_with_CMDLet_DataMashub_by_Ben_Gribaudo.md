---
title: "BI-TV #23: Редактирование кода скриптов Power Query в VS Code - решение Ben Gribaudo"
date: 2020-08-31
header:
  og_image: /assets/images/_23.png
categories:
  - Video
tags:
  - Power BI
---
<!-- markdownlint-disable MD040 MD013 -->
Редактирование кода скриптов Power Query в VS Code - решение Ben Gribaudo:
[https://bengribaudo.com/blog/2020/07/16/5356/editing-report-spreadsheet-mashups-in-vscode#more-5356](https://bengribaudo.com/blog/2020/07/16/5356/editing-report-spreadsheet-mashups-in-vscode#more-5356)

1. Устанавливаем VS Code [https://code.visualstudio.com/Download](https://code.visualstudio.com/Download). При установке ставим галку ["Add to Path"](https://s3-eu-central-1.amazonaws.com/nfd/PublicSharebales/addtopath-1598852869.png).

2. В VS Code устанавливаем расширение "Power Query / M Language" (ctrl+shift+P - установить расширение - вводим название).

3. Устанавливаем PowerShell Core 7 c github: [https://github.com/powershell/powershell#get-powershell](https://github.com/powershell/powershell#get-powershell).

4. Устанавливаем Data Mashup Cmdlet, для этого вводим PowerShell Core 7 команду:

    ```
    Install-Module -Name DataMashup -AllowPrerelease
    ```

    Если выдает ошибку - чтобы заработала возможность установки скриптов из галереи используем команду.

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    ```

    Решение найдено [здесь](https://www.myerrorsandmysolutions.com/unable-to-resolve-package-source-https-www-powershellgallery-com-api-v2/).

5. В скрипте функции заменяем code на путь до VS Code на вашем компьютере:

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
        Start-Process "code" "`"$($tempFile)`"" -Wait 
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

7. Создаем файл Profile, чтобы положить в него функцию, чтобы после перезапуска Power Shell она была доступна

    ```
    if (!(Test-Path -Path $PROFILE)) {
       New-Item -ItemType File -Path $PROFILE -Force
     }
    ```

8. Открываем файл Profile командой ниже, вставляем в notepad функцию из шага 6, сохраняем файл.

    ```
    notepad $PROFILE

    ```

9. Перезапускаем PowerShell 7

10. Редактируем код Power Query XLSX и PBI файлов в VS Code командой:

    ```
    Edit-DataMashup SomeFile.xlsx
    ```



<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='https://www.youtube.com/embed/XY7qf1wlgyUп' frameborder='0' allowfullscreen></iframe></div>
