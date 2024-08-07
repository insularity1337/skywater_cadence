Что нужно сделать чтобы использовать SkyWater 130nm бибилотеки с софтом от Cadence:
1. Замена всех `power_gating_pin("power_pin_1", 1.0000000000)` на `power_gating_pin("power_pin_1", 1)` во всех `.lib` файлах
2. Добавить `CUT` слой в технологические lef файлы ([то что в репозитори](https://github.com/google/skywater-pdk/issues/308) не соответствует руководству по LEF/DEF);
3. `.qrc` и `.captable` файлы не [поставляются](https://github.com/google/skywater-pdk/issues/187), нужно сгенерировать самостоятельно:
    ```bash
      generateCapTbl -ict skywater.ict -lef $SKYWATER_PDK/libraries/sky130_fd_sc_lp/latest/tech/sky130_fd_sc_lp.tlef -output $SKYWATER_PDK/libraries/sky130_fd_sc_lp/latest/lp.captbl
      ```
4. В некоторых сгенерированных `.lib` файлах без постфикса ccsnoise, может быть [лишняя информация](https://github.com/google/skywater-pdk/issues/77), её нужно удалить
5. Хоть и указано что в `sky130_fd_sc_lp` есть [clock-gating ячейки](https://skywater-pdk.readthedocs.io/en/main/contents/libraries/foundry-provided.html#sky130-fd-sc-lp-low-voltage-2-0v-low-power-standard-cell-library), но все они обозначены в библиотеке с атрибутом `dont_use`. Я менял значение атрибута в рамках синтеза
6. Чтобы иметь возможность использовать SRPG триггеры в рамках синтеза нужно изменить атрибуты соответствующих триггеров в `.lib` файлах например:
    ```
    ...
    cell ("sky130_fd_sc_lp__srsdfrtp_1") {
    ...
        pg_pin ("KAPWR") {
            pg_type: "internal_power";
    ...
        pin ("SLEEP_B") {
        ...
            power_gating_pin("power_pin_1", 1.0000000000);
        }
        power_gating_cell : "PG_1";
    ```
    Заменить на:
    ```
    ...
    cell ("sky130_fd_sc_lp__srsdfrtp_1") {
    ...
        pg_pin ("KAPWR") {
            pg_type: "backup_power";
    ...
        pin ("SLEEP_B") {
        ...
            power_gating_pin("save_restore", 1);
        }
        retention_cell : "PG_1";
    ```
7. Для `sky130_fd_sc_lp_lsbuf*` (и не только для них) [некорректно указан](https://github.com/google/skywater-pdk/issues/288) `related_bias_pin`, нужно менять руками
8. В ячейках изоляции `sky130_fd_sc_lp__iso*` в `.lib` файлах есть пин питания VGND, а в `.v` описаниях его нет, поэтому нет возможносто корректно промоделировать их работу
