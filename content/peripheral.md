# 4 框架外设驱动  
本章介绍基于框架的外设驱动。 
外设驱动位于/app/peripheral/ 目录下，外设驱动需要实现pando object的相关接口，会涉及该外设以及esp8266的驱动，这需要参考各自的数据手册和编程手册，esp8266的驱动放在driver目录下。      
esp8266的驱动有uart、SPI、PWM、I2C、adc，和通用GPIO。  

## 4.1 实现object的外设接口
一般是实现init、get，和set的功能。init对IO口进行定义和初始化，get即获取外设的状态，set需要执行指令，改变外设的状态。  
这里依然接着上一章节的RGB的例子，在这里讲述怎么实现外设驱动。  

## 4.1.1 引用相应的驱动文件  
驱动RGB需要使用PWM功能，所以头文件里面需要引用pwm文件。  
`#include "driver/pwm.h"`  

### 4.1.2 实现init功能  
对外设进行IO口定义和初始化，代码如下：  
```c
void ICACHE_FLASH_ATTR
peri_rgb_light_init(struct PWM_APP_PARAM light_param,struct PWM_INIT light_init)
{    
    spi_flash_write((PRIV_PARAM_START_SEC + RGB_LIGHT_PRIV_SAVE) * SPI_FLASH_SEC_SIZE,
    	    (uint32 *)&light_param, sizeof(struct PWM_APP_PARAM));
            
    pwm_init(light_param,light_init);
    pwm_start();    
}  
```

### 4.1.3 实现set功能  
传入新的值，更新pwm状态，代码如下：
```c  
void ICACHE_FLASH_ATTR 
peri_rgb_light_param_set( struct PWM_APP_PARAM light_param)
{
    pwm_set_freq(light_param.pwm_freq);
    pwm_set_duty(light_param.pwm_duty[0], 0); // red colour.
    pwm_set_duty(light_param.pwm_duty[1], 1); // green colour.
    pwm_set_duty(light_param.pwm_duty[2], 2); // blue colour.
    
    pwm_start();
    
    spi_flash_erase_sector(PRIV_PARAM_START_SEC + RGB_LIGHT_PRIV_SAVE);
	spi_flash_write((PRIV_PARAM_START_SEC + RGB_LIGHT_PRIV_SAVE) * SPI_FLASH_SEC_SIZE,
	    (uint32 *)&light_param, sizeof(struct PWM_APP_PARAM));
}
```

### 4.1.4 实现get功能  
返回当前外设的状态，实现代码如下：  

```c  
struct PWM_APP_PARAM ICACHE_FLASH_ATTR
peri_rgb_light_param_get(void)
{
    struct PWM_APP_PARAM ret;
    
    spi_flash_read((PRIV_PARAM_START_SEC + RGB_LIGHT_PRIV_SAVE) * SPI_FLASH_SEC_SIZE,
    	(uint32 *)&ret, sizeof(struct PWM_APP_PARAM));
    return ret;
}
```
