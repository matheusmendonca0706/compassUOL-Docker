Aqui está a versão completa utilizando a interface Runnable com a criação manual das Threads e divisão da imagem em blocos, sem usar o ExecutorService.
Pode copiar e colar no seu arquivo ImageMeanFilter.java da pasta concurrent:
```java
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import javax.imageio.ImageIO;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * This class provides functionality to apply a mean filter to an image.
 * The mean filter is used to smooth images by averaging the pixel values
 * in a neighborhood defined by a kernel size.
 * * <p>Usage example:</p>
 * <pre>
 * {@code
 * ImageMeanFilter.applyMeanFilter("input.jpg", "output.jpg", 3, 4);
 * }
 * </pre>
 * * <p>Supported image formats: JPG, PNG</p>
 */
public class ImageMeanFilter {
    
    /**
     * Applies mean filter to an image concurrently using Runnable
     * * @param inputPath  Path to input image
     * @param outputPath Path to output image 
     * @param kernelSize Size of mean kernel
     * @param numThreads Number of threads to use
     * @throws IOException If there is an error reading/writing
     */
    public static void applyMeanFilter(String inputPath, String outputPath, int kernelSize, int numThreads) throws IOException {
        BufferedImage originalImage = ImageIO.read(new File(inputPath));
        
        int width = originalImage.getWidth();
        int height = originalImage.getHeight();
        
        BufferedImage filteredImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
        
        AtomicInteger pixelsAlterados = new AtomicInteger(0);
        AtomicInteger pixelsInalterados = new AtomicInteger(0);

        Thread[] threads = new Thread[numThreads];
        int rowsPerThread = height / numThreads;

        for (int i = 0; i < numThreads; i++) {
            final int startY = i * rowsPerThread;
            // A última thread pega o resto das linhas caso a divisão não seja exata
            final int endY = (i == numThreads - 1) ? height : startY + rowsPerThread;

            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int y = startY; y < endY; y++) {
                        for (int x = 0; x < width; x++) {
                            int[] avgColor = calculateNeighborhoodAverage(originalImage, x, y, kernelSize);
                            
                            int newRgb = (avgColor[0] << 16) | (avgColor[1] << 8) | avgColor[2];
                            int originalRgb = originalImage.getRGB(x, y) & 0xFFFFFF;
                            
                            if (originalRgb != newRgb) {
                                pixelsAlterados.incrementAndGet();
                            } else {
                                pixelsInalterados.incrementAndGet();
                            }
                            
                            // BufferedImage.setRGB é thread-safe o suficiente para pixels distintos
                            filteredImage.setRGB(x, y, newRgb);
                        }
                    }
                }
            });
            threads[i].start();
        }
        
        // Aguarda todas as threads terminarem
        for (int i = 0; i < numThreads; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                System.err.println("Thread interrompida: " + e.getMessage());
                Thread.currentThread().interrupt();
            }
        }

        ImageIO.write(filteredImage, "jpg", new File(outputPath));
        
        System.out.println("Pixels alterados: " + pixelsAlterados.get());
        System.out.println("Pixels inalterados: " + pixelsInalterados.get());
    }
    
    /**
     * Calculates average colors in a pixel's neighborhood
     * * @param image      Source image
     * @param centerX    X coordinate of center pixel
     * @param centerY    Y coordinate of center pixel
     * @param kernelSize Kernel size
     * @return Array with R, G, B averages
     */
    private static int[] calculateNeighborhoodAverage(BufferedImage image, int centerX, int centerY, int kernelSize) {
        int width = image.getWidth();
        int height = image.getHeight();
        int pad = kernelSize / 2;
        
        long redSum = 0, greenSum = 0, blueSum = 0;
        int pixelCount = 0;
        
        for (int dy = -pad; dy <= pad; dy++) {
            for (int dx = -pad; dx <= pad; dx++) {
                int x = centerX + dx;
                int y = centerY + dy;
                
                if (x >= 0 && x < width && y >= 0 && y < height) {
                    int rgb = image.getRGB(x, y);
                    
                    int red = (rgb >> 16) & 0xFF;
                    int green = (rgb >> 8) & 0xFF;
                    int blue = rgb & 0xFF;
                    
                    redSum += red;
                    greenSum += green;
                    blueSum += blue;
                    pixelCount++;
                }
            }
        }
        
        return new int[] {
            (int)(redSum / pixelCount),
            (int)(greenSum / pixelCount),
            (int)(blueSum / pixelCount)
        };
    }
    
    /**
     * Main method for demonstration
     * * Usage: java ImageMeanFilter <input_file> <num_threads>
     */
    public static void main(String[] args) {
        if (args.length < 2) {
            System.err.println("Usage: java ImageMeanFilter <input_file> <num_threads>");
            System.exit(1);
        }

        String inputFile = args[0];
        int numThreads = Integer.parseInt(args[1]);

        if (numThreads < 2) {
            System.err.println("O número de threads deve ser no mínimo 2.");
            System.exit(1);
        }

        try {
            applyMeanFilter(inputFile, "filtered_output.jpg", 7, numThreads);
        } catch (IOException e) {
            System.err.println("Error processing image: " + e.getMessage());
        }
    }
}

```
