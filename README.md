# pgs-parser

JavaScript library for parsing subtitles in PGS format (`.sup` extension). Written with help from an excellent [blog post](https://blog.thescorpius.com/index.php/2017/07/15/presentation-graphic-stream-sup-files-bluray-subtitle-format/) on the format and from inspecting other parser implementations.

## Example Usage

```ts
import { DisplaySet, parseDisplaySets } from 'pgs-parser';

async function subtitleImageDataUrls(file: File): Promise<string[]> {
    const dataUrls: string[] = [];
    const canvas = document.createElement('canvas');
    let imageDataArray: Uint8ClampedArray | undefined;

    await file
        .stream()
        .pipeThrough(parseDisplaySets())
        .pipeTo(new WritableStream<DisplaySet>({
            write(displaySet, controller) {
                // Only display sets with object definition segments contain images
                if (displaySet.objectDefinitionSegments.length > 0) {
                    const screenWidth = displaySet.presentationCompositionSegment.width;
                    const screenHeight = displaySet.presentationCompositionSegment.height;

                    // Optionally provide a pre-allocated array to save memory
                    imageDataArray =
                        imageDataArray === undefined || imageDataArray.length < screenHeight * screenWidth * 4
                            ? new Uint8ClampedArray(screenWidth * screenHeight * 4)
                            : imageDataArray;
                    const imageData = displaySet.imageData(imageDataArray);

                    canvas.width = imageData.width;
                    canvas.height = imageData.height;
                    const context = canvas.getContext('2d')!;
                    context.putImageData(imageData, 0, 0);
                    dataUrls.push(canvas.toDataURL('image/png'));
                }
            }
        }));

    return dataUrls;
}
```