# Libraire / component pour LCD utilisant i2c

dans le CMAKE : ``` set(EXTRA_COMPONENT_DIRS ../LTG_lib/)```
de cette manière toutes les librairies / components sont importé

(librairies / components nons spimplifé)

```c
/****************************   include     ****************************/
#include <stdio.h>
#define LOG_LOCAL_LEVEL ESP_LOG_INFO
#include "esp_log.h"
#include "driver/gpio.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "lcd.h"
//#include "test.h"

/****************************   define     ****************************/


#define LCD_adress 0x27
#define LCD_colonne 16
#define LCD_ligne 2
#define LCD_SDA_pin 48
#define LCD_SCL_pin 2


/****************************   Variable global     ****************************/
static const char *TAG = "lcd_example";


/****************************   Fonction interne     ****************************/
static void initialise(void);
static void lcd_demo(void);

/****************************   Objet     ****************************/
lcd_handle_t lcd_handle = LCD_HANDLE_DEFAULT_CONFIG();




/****************************   main     ****************************/
void app_main(void)
{
    initialise();
    test();

    while (true)
    {
        ESP_LOGI(TAG, "Running LCD Demo");
        lcd_demo();
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

// Perform initilisation functions
static void initialise(void)
{
    i2c_config_t i2c_config = {
        .mode = I2C_MODE_MASTER,
        .sda_io_num = LCD_SDA_pin,
        .scl_io_num = LCD_SCL_pin,
        .sda_pullup_en = GPIO_PULLUP_ENABLE,
        .scl_pullup_en = GPIO_PULLUP_ENABLE,
        .master.clk_speed = I2C_MASTER_FREQ_HZ,
    };

    // Initialise i2c
    ESP_LOGD(TAG, "Installing i2c driver in master mode on channel %d", I2C_MASTER_NUM);
    ESP_ERROR_CHECK(i2c_driver_install(I2C_MASTER_NUM, I2C_MODE_MASTER, 0, 0, 0));
    ESP_LOGD(TAG,
             "Configuring i2c parameters.\n\tMode: %d\n\tSDA pin:%d\n\tSCL pin:%d\n\tSDA pullup:%d\n\tSCL pullup:%d\n\tClock speed:%.3fkHz",
             i2c_config.mode, i2c_config.sda_io_num, i2c_config.scl_io_num,
             i2c_config.sda_pullup_en, i2c_config.scl_pullup_en,
             i2c_config.master.clk_speed / 1000.0);
    ESP_ERROR_CHECK(i2c_param_config(I2C_MASTER_NUM, &i2c_config));

    // Modify default lcd_handle details
    lcd_handle.i2c_port = I2C_MASTER_NUM;
    lcd_handle.address = LCD_adress;
    lcd_handle.columns = LCD_colonne;
    lcd_handle.rows = LCD_ligne;
    lcd_handle.backlight = LCD_BACKLIGHT;

    // Initialise LCD
    ESP_ERROR_CHECK(lcd_init(&lcd_handle));

    return;
}

/**
 * @brief Demonstrate the LCD
 */
static void lcd_demo(void)
{

    lcd_clear_screen(&lcd_handle);
    lcd_write_str(&lcd_handle, "Hello world 6!");

    vTaskDelete(NULL);
}
```