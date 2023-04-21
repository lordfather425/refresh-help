$users = Get-ChildItem -Path "C:\Users\" | Where-Object { $_.PSIsContainer }

$selectedUser = $users | Out-GridView -Title "Select a user to copy folders from" -PassThru

if ($selectedUser) {
    $sourceFolders = @(
        Join-Path $selectedUser.FullName "Downloads"
        Join-Path $selectedUser.FullName "Documents"
        Join-Path $selectedUser.FullName "Desktop"
        Join-Path $selectedUser.FullName "Favorites"
        Join-Path $selectedUser.FullName "Pictures"
        Join-Path $selectedUser.FullName "AppData\Local\Google\Chrome\User Data\Default\Bookmarks"
    )

    $destFolder = "$env:userprofile\Desktop\$env:COMPUTERNAME"

    New-Item -ItemType Directory -Path $destFolder -ErrorAction SilentlyContinue | Out-Null

    ForEach ($folder in $sourceFolders) {
        $folderName = (Split-Path -Path $folder -Leaf)
        $destFolderPath = Join-Path -Path $destFolder -ChildPath $folderName
        if($folderName -eq "Bookmarks"){
            $destFolderPath = "$destFolder\Bookmarks"
        }
        if(Test-Path -Path $folder -PathType Container){
            Copy-Item -Path $folder -Destination $destFolderPath -Recurse -Force
        } elseif (Test-Path -Path $folder -PathType Leaf) {
            Copy-Item -Path $folder -Destination $destFolderPath -Force
        }
    }

    Write-Host "Folders copied to $destFolder"
} else {
    Write-Host "No user selected. Exiting script."
}
