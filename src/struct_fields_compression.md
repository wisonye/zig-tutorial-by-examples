# Struct fields compression

Watch the following video to know more about how memory layout works:

[Andrew Kelley - Practical DOD](https://vimeo.com/649009599)

Then try the following real examples:

### 1. Normal version

Bytes size is `6` and alignment is `1` (byte). It uses `6` bytes to store `6`
boolean values.

```c
const SensorStatus = struct {
    const Self = @This();

    power: bool,
    gps: bool,
    lidar: bool,
    temperature_sensor: bool,
    oil_sensor: bool,
    humidity_sensor: bool,

    pub fn print_sensor_status(self: *const Self) void {
        print("\n>>> [ SensorStatus ] {{\n\tPower: {}\n\tGps:{}\n\tLidar: {}\n\tTemperatur_sensor: {}\n\tOil_sensor: {}\n\tHumidity_sensor: {}\n}}", .{
            // self.version,
            self.power,
            self.gps,
            self.lidar,
            self.temperature_sensor,
            self.oil_sensor,
            self.humidity_sensor,
        });
    }
};
```

```bash
# >>> SensorStatus byte size: 6, alignment: 1
# >>> [ SensorStatus ] {
#         Power: false
#         Gps:false
#         Lidar: false
#         Temperatur_sensor: false
#         Oil_sensor: false
#         Humidity_sensor: false
# }⏎
```

### 2. Fields compression version

Bytes size is `1` and alignment is `1` (byte). It uses `1` bytes to store `6`
boolean values:)

```c
const SensorStatus2 = struct {
    const Self = @This();
    const power_shift_bit = 0;
    const gps_shift_bit = 1;
    const lidar_shift_bit = 2;
    const temperature_shift_bit = 3;
    const oil_shift_bit = 4;
    const humidity_shift_bit = 5;

    bits: u8,

    pub fn init(
        power_on: bool,
        gps_on: bool,
        lidar_on: bool,
        temperature_sensor_on: bool,
        oil_sensor_on: bool,
        humidity_sensor_on: bool,
    ) Self {
        var self = Self{ .bits = 0x00 };

        if (power_on) {
            self.toggle_power();
        }
        if (gps_on) {
            self.toggle_gps();
        }
        if (lidar_on) {
            self.toggle_lidar();
        }
        if (temperature_sensor_on) {
            self.toggle_temperature_sensor();
        }
        if (oil_sensor_on) {
            self.toggle_oil_sensor();
        }
        if (humidity_sensor_on) {
            self.toggle_humidity_sensor();
        }

        return self;
    }

    fn is_bit_on_u8(value: u8, bit: u8) bool {
        switch (bit) {
            0 => return value & 0x01 == 0x01,
            1 => return value & 0x02 == 0x02,
            2 => return value & 0x04 == 0x04,
            3 => return value & 0x08 == 0x08,
            4 => return value & 0x10 == 0x10,
            5 => return value & 0x20 == 0x20,
            6 => return value & 0x40 == 0x40,
            7 => return value & 0x80 == 0x80,
            else => return false,
        }
    }

    pub fn is_power_on(self: *const Self) bool {
        return is_bit_on_u8(self.bits, Self.power_shift_bit);
    }
    pub fn is_gps_on(self: *const Self) bool {
        return is_bit_on_u8(self.bits, Self.gps_shift_bit);
    }
    pub fn is_lidar_on(self: *const Self) bool {
        return is_bit_on_u8(self.bits, Self.lidar_shift_bit);
    }
    pub fn is_temperature_sensor_on(self: *const Self) bool {
        return is_bit_on_u8(self.bits, Self.temperature_shift_bit);
    }
    pub fn is_oil_sensor_on(self: *const Self) bool {
        return is_bit_on_u8(self.bits, Self.oil_shift_bit);
    }
    pub fn is_humidity_sensor_on(self: *const Self) bool {
        return is_bit_on_u8(self.bits, Self.humidity_shift_bit);
    }

    pub fn toggle_power(self: *Self) void {
        self.bits ^= (1 << Self.power_shift_bit);
    }
    pub fn toggle_gps(self: *Self) void {
        self.bits ^= (1 << Self.gps_shift_bit);
    }
    pub fn toggle_lidar(self: *Self) void {
        self.bits ^= (1 << Self.lidar_shift_bit);
    }
    pub fn toggle_temperature_sensor(self: *Self) void {
        self.bits ^= (1 << Self.temperature_shift_bit);
    }
    pub fn toggle_oil_sensor(self: *Self) void {
        self.bits ^= (1 << Self.oil_shift_bit);
    }
    pub fn toggle_humidity_sensor(self: *Self) void {
        self.bits ^= (1 << Self.humidity_shift_bit);
    }

    pub fn print_sensor_status(self: *const Self) void {
        print("\n>>> [ SensorStatus2 ] {{\n\tbits: {b:0>8}\n\tPower: {}\n\tGps:{}\n\tLidar: {}\n\tTemperatur_sensor: {}\n\tOil_sensor: {}\n\tHumidity_sensor: {}\n}}", .{
            self.bits,
            self.is_power_on(),
            self.is_gps_on(),
            self.is_lidar_on(),
            self.is_temperature_sensor_on(),
            self.is_oil_sensor_on(),
            self.is_humidity_sensor_on(),
        });
    }
};
```

```bash
# >>> SensorStatus2 byte size: 1, alignment: 1
# >>> [ SensorStatus2 ] {
#         bits: 00100001
#         Power: true
#         Gps:false
#         Lidar: false
#         Temperatur_sensor: false
#         Oil_sensor: false
#         Humidity_sensor: true
# }
```

</br>

