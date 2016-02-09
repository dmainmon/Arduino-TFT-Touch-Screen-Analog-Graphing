// Touch screen library with X Y and Z (pressure) readings as well
// Analog sensor data graph
// Code by Damon Borgnino

#include "TouchScreen.h"
#include <Adafruit_GFX.h>    // Core graphics library
#include <Adafruit_TFTLCD.h> // Hardware-specific library

#define DEBUG

#define LCD_CS A3
#define LCD_CD A2
#define LCD_WR A1
#define LCD_RD A0

#define LCD_RESET A4

// These are the pins for the shield!
#define YP A3  // must be an analog pin, use "An" notation!
#define XM A2  // must be an analog pin, use "An" notation!
#define YM 9   // can be a digital pin
#define XP 8   // can be a digital pin

#define MINPRESSURE 1
#define MAXPRESSURE 1000

// calibration mins and max for raw data when touching edges of screen
// YOU CAN USE THIS SKETCH TO DETERMINE THE RAW X AND Y OF THE EDGES TO GET YOUR HIGHS AND LOWS FOR X AND Y
#define TS_MINX 210
#define TS_MINY 210
#define TS_MAXX 915
#define TS_MAXY 910

#define BLACK   0x0000
#define BLUE    0x001F
#define RED     0xF800
#define GREEN   0x07E0
//#define CYAN    0x07FF
//#define MAGENTA 0xF81F
#define YELLOW  0xFFE0
#define WHITE   0xFFFF
//#define GREY      0xCE79
#define LIGHTGREY 0xDEDB


#define powerPin 53 //will turn on to power sensor(s)
#define sensorPin A14 // sensor one
#define sensor2Pin A15 // sensor two


const char* touchMeStr = " Analog Line Graph";

const int BASEVAL = 500; // Center graph baseline

Adafruit_TFTLCD tft(LCD_CS, LCD_CD, LCD_WR, LCD_RD, LCD_RESET);

TouchScreen ts = TouchScreen(XP, YP, XM, YM, 364);

int loopCounter = 0;
int pauseState = 0;
int menuState = 0;

int sensor1Val = 0; // used to hold sensor data
int sensor1ValOld = 0; // holds previous value for line graph
int sensor2Val = 0; // used t hold sensor 2 data
int sensor2ValOld = 0; // keeps old value needed to draw lines

// regions used for touch detection
int TopBtnArea[] = {0, 0, 0, 0};
int CenterBtnArea[] = {0, 0, 0, 0};
int BottomBtnArea[] = {0, 0, 0, 0};
int DataArea[] = {0, 0, 0, 0};
int DoneBtnArea[] = {220, 35, 65, 30};

// arrow buttons
int LeftArrowArea[] = { 33, 181, 41, 27}; // touch region for left arrow
int RightArrowArea[] = { 76, 181, 42, 27 }; // touch region for right arrow

// color buttons
int RedBtn[] = {40, 140, 35, 35};
int GreenBtn[] = {80, 140, 35, 35};
int YellowBtn[] = {120, 140, 35, 35};
int BlueBtn[] = {160, 140, 35, 35};
int WhiteBtn[] = {200, 140, 35, 35};
int BlackBtn[] = {240, 140, 35, 35};

int TopGraphColor = RED;
int CenterGraphColor = WHITE;
int BottomGraphColor = YELLOW;

int TopDirection = 1;  // 0=right, 1=left
int CenterDirection = 0;
int BottomDirection = 0;
/////////////////////////////////////////////////////////////////////////////////////////////

void setup(void) {

#ifdef DEBUG
  Serial.begin(9600);
#endif // DEBUG

  pinMode(powerPin, OUTPUT);
  digitalWrite(powerPin, HIGH); // turn sensor power on

  tft.reset();

  uint16_t identifier = tft.readID();

  tft.begin(identifier);

  tft.fillScreen(BLACK);
  tft.setRotation(1);
  tft.setCursor(30, 100);
  tft.setTextColor(RED);  tft.setTextSize(2);
  tft.println("LCD driver chip: ");
  tft.setCursor(100, 150);
  tft.setTextColor(BLUE);
  tft.println(identifier, HEX);

  delay(500);

  tft.fillScreen(BLACK);
  tft.setTextColor(YELLOW);

  tft.setCursor(0, 0);
  tft.println(touchMeStr);


}


//////////////////////////////////////////////////////////////////////////////////


void loop(void) {


  sensor1Val = analogRead(sensorPin); // read sensor data
  sensor2Val = analogRead(sensor2Pin); // read sensor data

  int audioVal = sensor2Val; // raw

  if (audioVal < BASEVAL)
    audioVal = (BASEVAL - audioVal) + BASEVAL;

  // a point object holds x y and z coordinates
  TSPoint p = ts.getPoint();

  pinMode(XM, OUTPUT);
  pinMode(YP, OUTPUT);

  // we have some minimum pressure we consider 'valid'
  // pressure of 0 means no pressing!
  if (p.z > MINPRESSURE && p.z < MAXPRESSURE) {

    //////////////////////////////////////////////////////////////////////////
    // because of rotation x is traNsposed with and y and height with width
    // this is typically the cause of cordinates being reversed
    //////////////////////////////////////////////////////////////////////////
    int   YY =  tft.height() - (map(p.x, TS_MINX, TS_MAXX, 0, tft.height()));
    int   XX =  tft.width() - (map(p.y, TS_MINY, TS_MAXY, 0, tft.width()));

    // flip the pause state on/off
    if (pauseState == 0)
    {
      pauseState = 1;
    }
    else // already in pause state
    {

      checkMenuPress( XX, YY);

    }


    // PRINTS OUT THE RAW AND TRANSPOSED DATA useful for troubleshooting
    if (pauseState == 1)
    {

      showMenuOptions();
      if (menuState == 0)
        showData(p.x, p.y, p.z, XX, YY);

      ///////////////////////////////////////////////////////////////////////////
#ifdef DEBUG

      // serial data
      Serial.print("Raw X = "); Serial.print(p.x);
      Serial.print(" Raw Y = "); Serial.println(p.y);
      Serial.print("Pressure = "); Serial.println(p.z);
      Serial.print("Screen XX: " );
      Serial.print(XX);
      Serial.print("  Screen YY:");
      Serial.println(YY);
      Serial.print("Raw Sensor 1 = "); Serial.println(sensor1Val);
      Serial.print("Raw Sensor 2 = "); Serial.println(sensor2Val);
      Serial.print("Loop counter "); Serial.println(loopCounter);

#endif // DEBUG ///////////////////////////////////////////////////////////

      delay(500);
    }
    else
    {

      // or reset the whole screen
      tft.fillScreen(BLACK);
      loopCounter = 0;

    }

  }
  else // if there is no touch event do this
  {

    if (pauseState == 0) // start / stop drawing data on touch
    {
      // the raw value can be used, but this can range from 0-1023 so some adjustments may be needed
      // here to make the graph more readable, otherwise values will easily go off the screen
      int sensor1Transposed = sensor1Val;

      // the value for this sensor is different so it needs to be adjusted to fit nice on the display
      // the more division used the less granular the data becomes
      // the raw data was too variable and higher than the other sensor
      // so to get them to fit together nicely, one or both sensor needs to be adjusted.

      int sensor2Transposed = (sensor2Val - tft.height()) / 2 ;


      // draw top graph
      if (TopDirection == 0)
        tft.drawLine(loopCounter, sensor1Transposed, loopCounter, 0, TopGraphColor);
      else
        tft.drawLine(tft.width() - loopCounter, sensor1Transposed, tft.width() - loopCounter, 0, TopGraphColor);


      //draw center graph
      if (CenterDirection == 0)
        tft.drawLine(loopCounter, sensor2ValOld, loopCounter + 1, sensor2Transposed, CenterGraphColor);
      else
        tft.drawLine(tft.width() - loopCounter + 1, sensor2ValOld, tft.width() - loopCounter, sensor2Transposed, CenterGraphColor);

      sensor2ValOld = sensor2Transposed;
      sensor1ValOld = sensor1Transposed;


      // draw bottom graph == 0)
      if (BottomDirection == 0)
        tft.drawLine( loopCounter, tft.height() - (audioVal - BASEVAL), loopCounter, tft.height(), BottomGraphColor);
      else
        tft.drawLine(tft.width() - loopCounter, tft.height() - (audioVal - BASEVAL), tft.width() - loopCounter, tft.height(), BottomGraphColor);


      if (loopCounter == tft.width()  )
      {
        loopCounter = 0;
        tft.fillScreen(BLACK);
      }
      else
      {
        loopCounter = loopCounter + 1;
      }

    }

  }

  delay(20); // helps smooth out graphs especially if more than one is drawing
}


////////////////////////////////////////////////////////////////////////////////////////////////


void checkMenuPress(int XX, int YY)
{

  if (XX >= TopBtnArea[0] && XX <= TopBtnArea[0] + TopBtnArea[2] &&
      YY >= TopBtnArea[1] && YY <= TopBtnArea[1] + TopBtnArea[3] )
  {
    menuState = 1;
  }
  else if (XX >= CenterBtnArea[0] && XX <= CenterBtnArea[0] + CenterBtnArea[2] &&
           YY >= CenterBtnArea[1] && YY <= CenterBtnArea[1] + CenterBtnArea[3] )
  {
    menuState = 2;
  }
  else if (XX >= BottomBtnArea[0] && XX <= BottomBtnArea[0] + BottomBtnArea[2] &&
           YY >= BottomBtnArea[1] && YY <= BottomBtnArea[1] + BottomBtnArea[3] )
  {
    menuState = 3;
  }

  else if (XX >= DoneBtnArea[0] && XX <= DoneBtnArea[0] + DoneBtnArea[2] &&
           YY >= DoneBtnArea[1] && YY <= DoneBtnArea[1] + DoneBtnArea[3] )
  {

    menuState = 0;
    pauseState = 0;

  }



  switch (menuState)
  {

    case 1: // top graph  direction

      if (XX >= LeftArrowArea[0] && XX <= LeftArrowArea[0] + LeftArrowArea[2] &&
          YY >= LeftArrowArea[1] && YY <= LeftArrowArea[1] + LeftArrowArea[3] )
      {
        TopDirection = 1;
        tft.drawRect(LeftArrowArea[0], LeftArrowArea[1], LeftArrowArea[2], LeftArrowArea[3], GREEN);
      }

      if (XX >= RightArrowArea[0] && XX <= RightArrowArea[0] + RightArrowArea[2] &&
          YY >= RightArrowArea[1] && YY <= RightArrowArea[1] + RightArrowArea[3] )
      {
        TopDirection = 0;
        tft.drawRect(RightArrowArea[0], RightArrowArea[1], RightArrowArea[2], RightArrowArea[3], GREEN);
      }

      //////////////////////////
      //top graph colors

      if (XX >= RedBtn[0] && XX <= RedBtn[0] + RedBtn[2] &&
          YY >= RedBtn[1] && YY <= RedBtn[1] + RedBtn[3] )
      {
        TopGraphColor = RED;
        tft.fillRect(RedBtn[0], RedBtn[1], RedBtn[2], RedBtn[3], RED);
      }
      else if (XX >= GreenBtn[0] && XX <= GreenBtn[0] + GreenBtn[2] &&
               YY >= GreenBtn[1] && YY <= GreenBtn[1] + GreenBtn[3] )
      {
        TopGraphColor = GREEN;
        tft.fillRect(GreenBtn[0], GreenBtn[1], GreenBtn[2], GreenBtn[3], GREEN);
      }
      else if (XX >= YellowBtn[0] && XX <= YellowBtn[0] + YellowBtn[2] &&
               YY >= YellowBtn[1] && YY <= YellowBtn[1] + YellowBtn[3] )
      {
        TopGraphColor = YELLOW;
        tft.fillRect(YellowBtn[0], YellowBtn[1], YellowBtn[2], YellowBtn[3], YELLOW);
      }
      else if (XX >= BlueBtn[0] && XX <= BlueBtn[0] + BlueBtn[2] &&
               YY >= BlueBtn[1] && YY <= BlueBtn[1] + BlueBtn[3] )
      {
        TopGraphColor = BLUE;
        tft.fillRect(BlueBtn[0], BlueBtn[1], BlueBtn[2], BlueBtn[3], BLUE);
      }
      else if (XX >= WhiteBtn[0] && XX <= WhiteBtn[0] + WhiteBtn[2] &&
               YY >= WhiteBtn[1] && YY <= WhiteBtn[1] + WhiteBtn[3] )
      {
        TopGraphColor = WHITE;
        tft.fillRect(WhiteBtn[0], WhiteBtn[1], WhiteBtn[2], WhiteBtn[3], WHITE);
      }
      else if (XX >= BlackBtn[0] && XX <= BlackBtn[0] + BlackBtn[2] &&
               YY >= BlackBtn[1] && YY <= BlackBtn[1] + BlackBtn[3] )
      {
        TopGraphColor = BLACK;
        tft.fillRect(BlackBtn[0], BlackBtn[1], BlackBtn[2], BlackBtn[3], BLACK);
      }

      break;

    case 2: // center graph

      // center graph arrows
      if (XX >= LeftArrowArea[0] && XX <= LeftArrowArea[0] + LeftArrowArea[2] &&
          YY >= LeftArrowArea[1] && YY <= LeftArrowArea[1] + LeftArrowArea[3] )
      {
        CenterDirection = 1;
        tft.drawRect(LeftArrowArea[0], LeftArrowArea[1], LeftArrowArea[2], LeftArrowArea[3], GREEN);
      }

      if (XX >= RightArrowArea[0] && XX <= RightArrowArea[0] + RightArrowArea[2] &&
          YY >= RightArrowArea[1] && YY <= RightArrowArea[1] + RightArrowArea[3] )
      {
        CenterDirection = 0;
        tft.drawRect(RightArrowArea[0], RightArrowArea[1], RightArrowArea[2], RightArrowArea[3], GREEN);
      }

      ////////////////
      // center graph colors


      if (XX >= RedBtn[0] && XX <= RedBtn[0] + RedBtn[2] &&
          YY >= RedBtn[1] && YY <= RedBtn[1] + RedBtn[3] )
      {
        CenterGraphColor = RED;
        tft.fillRect(RedBtn[0], RedBtn[1], RedBtn[2], RedBtn[3], RED);
      }
      else if (XX >= GreenBtn[0] && XX <= GreenBtn[0] + GreenBtn[2] &&
               YY >= GreenBtn[1] && YY <= GreenBtn[1] + GreenBtn[3] )
      {
        CenterGraphColor = GREEN;
        tft.fillRect(GreenBtn[0], GreenBtn[1], GreenBtn[2], GreenBtn[3], GREEN);
      }
      else if (XX >= YellowBtn[0] && XX <= YellowBtn[0] + YellowBtn[2] &&
               YY >= YellowBtn[1] && YY <= YellowBtn[1] + YellowBtn[3] )
      {
        CenterGraphColor = YELLOW;
        tft.fillRect(YellowBtn[0], YellowBtn[1], YellowBtn[2], YellowBtn[3], YELLOW);
      }
      else if (XX >= BlueBtn[0] && XX <= BlueBtn[0] + BlueBtn[2] &&
               YY >= BlueBtn[1] && YY <= BlueBtn[1] + BlueBtn[3] )
      {
        CenterGraphColor = BLUE;
        tft.fillRect(BlueBtn[0], BlueBtn[1], BlueBtn[2], BlueBtn[3], BLUE);
      }
      else if (XX >= WhiteBtn[0] && XX <= WhiteBtn[0] + WhiteBtn[2] &&
               YY >= WhiteBtn[1] && YY <= WhiteBtn[1] + WhiteBtn[3] )
      {
        CenterGraphColor = WHITE;
        tft.fillRect(WhiteBtn[0], WhiteBtn[1], WhiteBtn[2], WhiteBtn[3], WHITE);
      }
      else if (XX >= BlackBtn[0] && XX <= BlackBtn[0] + BlackBtn[2] &&
               YY >= BlackBtn[1] && YY <= BlackBtn[1] + BlackBtn[3] )
      {
        CenterGraphColor = BLACK;
        tft.fillRect(BlackBtn[0], BlackBtn[1], BlackBtn[2], BlackBtn[3], BLACK);
      }
      break;

    case 3:
      // bottom graph arrows

      if (XX >= LeftArrowArea[0] && XX <= LeftArrowArea[0] + LeftArrowArea[2] &&
          YY >= LeftArrowArea[1] && YY <= LeftArrowArea[1] + LeftArrowArea[3] )
      {
        BottomDirection = 1;
        tft.drawRect(LeftArrowArea[0], LeftArrowArea[1], LeftArrowArea[2], LeftArrowArea[3], GREEN);
      }

      if (XX >= RightArrowArea[0] && XX <= RightArrowArea[0] + RightArrowArea[2] &&
          YY >= RightArrowArea[1] && YY <= RightArrowArea[1] + RightArrowArea[3] )
      {
        BottomDirection = 0;
        tft.drawRect(RightArrowArea[0], RightArrowArea[1], RightArrowArea[2], RightArrowArea[3], GREEN);
      }


      /////////////////////
      // bottom graph colors

      if (XX >= RedBtn[0] && XX <= RedBtn[0] + RedBtn[2] &&
          YY >= RedBtn[1] && YY <= RedBtn[1] + RedBtn[3] )
      {
        BottomGraphColor = RED;
        tft.fillRect(RedBtn[0], RedBtn[1], RedBtn[2], RedBtn[3], RED);
      }
      else if (XX >= GreenBtn[0] && XX <= GreenBtn[0] + GreenBtn[2] &&
               YY >= GreenBtn[1] && YY <= GreenBtn[1] + GreenBtn[3] )
      {
        BottomGraphColor = GREEN;
        tft.fillRect(GreenBtn[0], GreenBtn[1], GreenBtn[2], GreenBtn[3], GREEN);
      }
      else if (XX >= YellowBtn[0] && XX <= YellowBtn[0] + YellowBtn[2] &&
               YY >= YellowBtn[1] && YY <= YellowBtn[1] + YellowBtn[3] )
      {
        BottomGraphColor = YELLOW;
        tft.fillRect(YellowBtn[0], YellowBtn[1], YellowBtn[2], YellowBtn[3], YELLOW);
      }
      else if (XX >= BlueBtn[0] && XX <= BlueBtn[0] + BlueBtn[2] &&
               YY >= BlueBtn[1] && YY <= BlueBtn[1] + BlueBtn[3] )
      {
        BottomGraphColor = BLUE;
        tft.fillRect(BlueBtn[0], BlueBtn[1], BlueBtn[2], BlueBtn[3], BLUE);
      }
      else if (XX >= WhiteBtn[0] && XX <= WhiteBtn[0] + WhiteBtn[2] &&
               YY >= WhiteBtn[1] && YY <= WhiteBtn[1] + WhiteBtn[3] )
      {
        BottomGraphColor = WHITE;
        tft.fillRect(WhiteBtn[0], WhiteBtn[1], WhiteBtn[2], WhiteBtn[3], WHITE);
      }
      else if (XX >= BlackBtn[0] && XX <= BlackBtn[0] + BlackBtn[2] &&
               YY >= BlackBtn[1] && YY <= BlackBtn[1] + BlackBtn[3] )
      {
        BottomGraphColor = BLACK;
        tft.fillRect(BlackBtn[0], BlackBtn[1], BlackBtn[2], BlackBtn[3], BLACK);
      }
      break;

  }

}

//////////////////////////////////////////////////////////////////////////////////////////////////


void showMenuOptions() // shows buttons and sets options
{

  int borderSize = 30; // pixels
  int marginSize = 10;

  tft.setTextColor( YELLOW);
  tft.fillRect(borderSize, borderSize,  tft.width() - (borderSize * 2), tft.height() - (borderSize * 2), LIGHTGREY);

  tft.drawRect(borderSize, borderSize,  tft.width() - (borderSize * 2), tft.height() - (borderSize * 2), RED);
  tft.setCursor((tft.width() / 2) - borderSize,  borderSize + marginSize);
  tft.setTextSize(3);
  tft.println("Menu");

  int indentRight = 40;
  int indentTop = 60; // where to start the text block
  int lineHeight = 30;
  int buttonSize = 83;

  tft.setCursor(  indentRight, indentTop = indentTop + 20);

  // set the button region so a press can be determined
  TopBtnArea[0] = indentRight - 5;
  TopBtnArea[1] = indentTop - 5;
  TopBtnArea[2] = buttonSize;
  TopBtnArea[3] = lineHeight + 7;



  if (menuState == 1) // top
  {
    // change button
    tft.fillRect(TopBtnArea[0], TopBtnArea[1], TopBtnArea[2], TopBtnArea[3], GREEN);
    tft.drawRect(TopBtnArea[0], TopBtnArea[1], TopBtnArea[2], TopBtnArea[3], RED);
    setDirectionButtons();
    setColorOptions();
    fillCurrentColor(TopGraphColor);

  }

  else
  {
    //reset button
    tft.fillRect(TopBtnArea[0], TopBtnArea[1], TopBtnArea[2], TopBtnArea[3], WHITE);
    tft.drawRect(TopBtnArea[0], TopBtnArea[1], TopBtnArea[2], TopBtnArea[3], RED);
  }
  tft.setTextSize(2);
  tft.setTextColor( BLACK);
  tft.setCursor( indentRight + 5, indentTop + 7 );
  tft.print(" TOP ");

  CenterBtnArea[0] = indentRight - 5 + buttonSize;
  CenterBtnArea[1] = indentTop - 5;
  CenterBtnArea[2] = buttonSize;
  CenterBtnArea[3] = lineHeight + 7;

  if (menuState == 2)
  {

    tft.fillRect(CenterBtnArea[0], CenterBtnArea[1], CenterBtnArea[2], CenterBtnArea[3], GREEN);
    tft.drawRect(CenterBtnArea[0], CenterBtnArea[1], CenterBtnArea[2], CenterBtnArea[3], RED);
    setDirectionButtons();
    setColorOptions();
    fillCurrentColor(CenterGraphColor);
  }
  else
  {
    tft.fillRect(CenterBtnArea[0], CenterBtnArea[1], CenterBtnArea[2], CenterBtnArea[3], WHITE);
    tft.drawRect(CenterBtnArea[0], CenterBtnArea[1], CenterBtnArea[2], CenterBtnArea[3], RED);

  }

  tft.setTextSize(2);
  tft.setTextColor( BLACK);
  tft.setCursor( indentRight + buttonSize, indentTop + 7 );
  tft.print("CENTER");

  BottomBtnArea[0] = indentRight - 5 + (buttonSize * 2);
  BottomBtnArea[1] = indentTop - 5;
  BottomBtnArea[2] = buttonSize;
  BottomBtnArea[3] = lineHeight + 7;


  if (menuState == 3)
  {
    tft.fillRect(BottomBtnArea[0], BottomBtnArea[1], BottomBtnArea[2], BottomBtnArea[3], GREEN);
    tft.drawRect(BottomBtnArea[0], BottomBtnArea[1], BottomBtnArea[2], BottomBtnArea[3], RED);
    setDirectionButtons();
    setColorOptions();
    fillCurrentColor(BottomGraphColor);
  }
  else
  {
    tft.fillRect(BottomBtnArea[0], BottomBtnArea[1], BottomBtnArea[2], BottomBtnArea[3], WHITE);
    tft.drawRect(BottomBtnArea[0], BottomBtnArea[1], BottomBtnArea[2], BottomBtnArea[3], RED);
  }
  tft.setTextSize(2);
  tft.setTextColor( BLACK);
  tft.setCursor( indentRight + (buttonSize * 2), indentTop + 7 );
  tft.println("BOTTOM");


  drawDoneButton();

}

void fillCurrentColor(int color)
{
  switch (color)
  {
    case RED:
      tft.fillRect(RedBtn[0], RedBtn[1], RedBtn[2], RedBtn[3], RED);
      break;
    case GREEN:
      tft.fillRect(GreenBtn[0], GreenBtn[1], GreenBtn[2], GreenBtn[3], GREEN);
      break;
    case YELLOW:
      tft.fillRect(YellowBtn[0], YellowBtn[1], YellowBtn[2], YellowBtn[3], YELLOW);
      break;
    case BLUE:
      tft.fillRect(BlueBtn[0], BlueBtn[1], BlueBtn[2], BlueBtn[3], BLUE);
      break;
    case WHITE:
      tft.fillRect(WhiteBtn[0], WhiteBtn[1], WhiteBtn[2], WhiteBtn[3], WHITE);
      break;
    case BLACK:
      tft.fillRect(BlackBtn[0], BlackBtn[1], BlackBtn[2], BlackBtn[3], BLACK);
      break;
  }
}


void setDirectionButtons()
{


  switch (menuState)
  {
    case 1:
      if (TopDirection == 1)
        tft.drawRect(LeftArrowArea[0], LeftArrowArea[1], LeftArrowArea[2], LeftArrowArea[3], GREEN); // touch region for left arrow
      else
        tft.drawRect(RightArrowArea[0], RightArrowArea[1], RightArrowArea[2], RightArrowArea[3], GREEN); // touch region for left arrow

      break;

    case 2:
      if (CenterDirection == 1)
        tft.drawRect(LeftArrowArea[0], LeftArrowArea[1], LeftArrowArea[2], LeftArrowArea[3], GREEN); // touch region for left arrow
      else
        tft.drawRect(RightArrowArea[0], RightArrowArea[1], RightArrowArea[2], RightArrowArea[3], GREEN); // touch region for left arrow

      break;

    case 3:
      if (BottomDirection == 1)
        tft.drawRect(LeftArrowArea[0], LeftArrowArea[1], LeftArrowArea[2], LeftArrowArea[3], GREEN); // touch region for left arrow
      else
        tft.drawRect(RightArrowArea[0], RightArrowArea[1], RightArrowArea[2], RightArrowArea[3], GREEN); // touch region for left arrow

      break;
  }
  tft.fillRect(50, 191, 20, 9, RED);
  tft.fillTriangle(35, 195, 50, 185, 50, 205, RED);

  // tft.drawRect(76, 181, 42, 27, BLUE); // touch region for right arrow
  tft.fillRect(80, 191, 20, 9, GREEN);
  tft.fillTriangle(115, 195, 100, 185, 100, 205, GREEN);


}

void setColorOptions()
{

  tft.setCursor(DataArea[0], DataArea[1]);
  tft.setTextSize(1);
  tft.setTextColor(YELLOW);
  tft.println("Color");


  tft.drawRect(RedBtn[0], RedBtn[1], RedBtn[2], RedBtn[3], RED);
  tft.drawRect(GreenBtn[0], GreenBtn[1], GreenBtn[2], GreenBtn[3], GREEN);
  tft.drawRect(YellowBtn[0], YellowBtn[1], YellowBtn[2], YellowBtn[3], YELLOW);
  tft.drawRect(BlueBtn[0], BlueBtn[1], BlueBtn[2], BlueBtn[3], BLUE);
  tft.drawRect(WhiteBtn[0], WhiteBtn[1], WhiteBtn[2], WhiteBtn[3], WHITE);
  tft.drawRect(BlackBtn[0], BlackBtn[1], BlackBtn[2], BlackBtn[3], BLACK);

}


void showData(int x, int y, int z, int XX, int YY)
{
  int indentRight = 40;
  int indentTop = 125; // where to start the text block
  int lineHeight = 9;

  DataArea[0] = indentRight - 5;
  DataArea[1] = indentTop;
  DataArea[2] = 250;
  DataArea[3] = 80;
  
  tft.drawRect(DataArea[0], DataArea[1], DataArea[2], DataArea[3], RED);

  tft.setTextSize(1);
  tft.setTextColor( YELLOW); // converts the pressure reading to a color
  tft.setCursor(  indentRight, indentTop = indentTop + lineHeight);

  tft.println("DATA CAPTURE");
  tft.setCursor(  indentRight, indentTop = indentTop + lineHeight);

  tft.setTextColor( WHITE);
  tft.print("Raw X = "); tft.print(x);
  tft.print(" Raw Y = "); tft.println(y);
  tft.setCursor(  indentRight, indentTop = indentTop + lineHeight);

  tft.print("Pressure = "); tft.println(z);
  tft.setCursor(  indentRight, indentTop = indentTop + lineHeight);

  tft.print("ScreenX: " ); tft.print(XX);
  tft.print("  ScreenY:"); tft.println(YY);
  tft.setCursor(  indentRight, indentTop = indentTop + lineHeight);

  tft.print("Raw Sensor 1 "); tft.println(sensor1Val);
  tft.setCursor(  indentRight, indentTop = indentTop + lineHeight);

  tft.print("Raw Sensor 2 "); tft.println(sensor2Val);
  tft.setCursor(  indentRight, indentTop = indentTop + lineHeight);

  tft.print("Loop counter "); tft.println(loopCounter);
  tft.setCursor(  indentRight, indentTop = indentTop + lineHeight);
}

//////////////////////

void drawDoneButton()
{
  tft.fillRect(DoneBtnArea[0], DoneBtnArea[1], DoneBtnArea[2], DoneBtnArea[3], BLACK);
  tft.drawRect(DoneBtnArea[0], DoneBtnArea[1], DoneBtnArea[2], DoneBtnArea[3], WHITE);

  tft.setCursor(DoneBtnArea[0] + 10, DoneBtnArea[1] + 7);
  tft.setTextColor(WHITE);
  tft.print("Done");
}
/////////////////////////////////////////////////////////////////////////////////////////////////////
