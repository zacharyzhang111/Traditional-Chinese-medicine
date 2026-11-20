# Traditional-Chinese-medicine
*A product by arduino

I developed an Arduino-based image display system using an ST7789 TFT screen and an SD card. By integrating a push button with the Adafruit ImageReader library, I enabled users to switch between three preloaded images through physical interaction. The system translates simple button inputs into real-time visual changes, creating an intuitive connection between physical controls and digital content.

## Arduino Code

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_ST7789.h>
#include <SdFat.h>
#include <Adafruit_SPIFlash.h>
#include <Adafruit_ImageReader.h>

// Settings
const uint8_t sdCs = 7;
const uint8_t tftCs = 10;
const uint8_t tftDc = 9;
const uint8_t tftRst = 8;
const uint8_t buttonPin = 3;

// SD Mode
#define sdMode

#if defined(sdMode)
SdFat sdCard;
Adafruit_ImageReader imageReader(sdCard);
#else
// Flash Mode
#endif

// Display Object
Adafruit_ST7789 tftDisplay = Adafruit_ST7789(tftCs, tftDc, tftRst);

// Image Index
int currentIndex = 0;

// Image Output
void showImage() {
  ImageReturnCode imageStatus;

  tftDisplay.fillScreen(ST77XX_BLACK);

  if (currentIndex == 0) {
    Serial.println("Showing 1.bmp");
    imageStatus = imageReader.drawBMP("/1.bmp", tftDisplay, 0, 0);
  }
  else if (currentIndex == 1) {
    Serial.println("Showing 2.bmp");
    imageStatus = imageReader.drawBMP("/2.bmp", tftDisplay, 0, 0);
  }
  else if (currentIndex == 2) {
    Serial.println("Showing 3.bmp");
    imageStatus = imageReader.drawBMP("/3.bmp", tftDisplay, 0, 0);
  }

  // Load Check
  if (imageStatus != IMAGE_SUCCESS) {
    Serial.print("Error loading image, code: ");
    Serial.println(imageStatus);
  }
  else {
    Serial.println("Display OK");
  }
}

// Initialization
void setup() {

  // Serial Setup
  Serial.begin(9600);
  pinMode(buttonPin, INPUT);

  // TFT Setup
  tftDisplay.init(240, 320);
  tftDisplay.fillScreen(ST77XX_BLACK);
  tftDisplay.setRotation(2);

  // SD Setup
  if (!sdCard.begin(sdCs, SD_SCK_MHZ(10))) {
    Serial.println("SD card init failed!");
    while (1);
  }

  Serial.println("SD card ready.");

  // First Image
  showImage();

  Serial.println("Press button to switch image.");
}

// Main Loop
void loop() {

  // Button Scan
  int buttonState = digitalRead(buttonPin);

  // Button Press
  if (buttonState == LOW) {

    // Debounce
    delay(20);

    if (digitalRead(buttonPin) == LOW) {

      Serial.println("Button pressed!");

      // Image Switch
      currentIndex++;

      if (currentIndex > 2) {
        currentIndex = 0;
      }

      showImage();

      // Release Wait
      while (digitalRead(buttonPin) == LOW);

      delay(20);
    }
  }
}
```
