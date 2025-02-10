# Set the drive where the images are located
$drivePath = "D:\"

# Define the types of image files to check (e.g., jpg, jpeg, png, gif, bmp, etc.)
$imageTypes = @("*.jpg", "*.jpeg", "*.png", "*.gif", "*.bmp")

# Log file for broken images
$logFile = "D:\broken_images.log"

# Clear previous log if exists
if (Test-Path $logFile) {
    Remove-Item $logFile
}

# Initialize the counters
$totalFiles = 0
$processedFiles = 0

# Count total image files to process
foreach ($type in $imageTypes) {
    $totalFiles += (Get-ChildItem -Path $drivePath -Recurse -Filter $type).Count
}

# Loop through each image type
foreach ($type in $imageTypes) {
    # Get all image files of the current type
    Get-ChildItem -Path $drivePath -Recurse -Filter $type | ForEach-Object {
        $filePath = $_.FullName

        # Try to repair the image using ImageMagick (convert with strip to remove metadata)
        try {
            # Check the file with ImageMagick
            $outputFile = "$($filePath.Substring(0, $filePath.LastIndexOf('.')))_repaired$($_.Extension)"
            
            # Attempt to repair the image (use -strip to remove metadata and force conversion)
            & "C:\Program Files\ImageMagick-7.0.11-Q16\convert.exe" "$filePath" -strip "$outputFile"

            # If successful, remove the original broken file and rename the repaired one
            if (Test-Path $outputFile) {
                Remove-Item $filePath
                Rename-Item $outputFile $filePath
            }
        }
        catch {
            # If there's an error, log the broken image
            "$filePath is broken and could not be repaired." | Out-File -Append $logFile
        }

        # Increment the processed files counter
        $processedFiles++

        # Display progress
        Write-Host "Processed $processedFiles of $totalFiles files. ($($processedFiles / $totalFiles * 100)%)"
    }
}

Write-Host "Image repair process complete. Check the log file for broken images: $logFile"
