idf_component_register(
    SRCS "main.c" "rpm.c"
    INCLUDE_DIRS "."
    REQUIRES esp_driver_gpio esp_timer esp_common
)
