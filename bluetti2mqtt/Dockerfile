ARG BUILD_FROM
FROM $BUILD_FROM

# Copy data for add-on
COPY rootfs /

# Install fork bluetti_mqtt version
RUN apk add --no-cache git \
    && pip install --no-cache-dir git+https://github.com/semitop7/bluetti_mqtt.git \
    && chmod a+x /run.sh

# Apply grid_charge_amps patch to ALL devices
RUN for device in ac500 ac300 ac200L ac60 ep500 ep500p ep600 eb3a; do \
    sed -i "s|self.struct.add_uint_field('battery_range_end'|self.struct.add_uint_field('grid_charge_amps', 3019)\n        self.struct.add_uint_field('battery_range_end'|" \
    /usr/local/lib/python3.14/site-packages/bluetti_mqtt/core/devices/$device.py 2>/dev/null || true; \
  done

# Apply dc_input_power2 and dc_input_voltage2 patch to AC500
RUN sed -i "s|self.struct.add_decimal_field('internal_dc_input_current', 88, 1)|self.struct.add_decimal_field('internal_dc_input_current', 88, 1)\n        self.struct.add_decimal_field('dc_input_voltage2', 170, 1)\n        self.struct.add_uint_field('dc_input_power2', 172)|" \
    /usr/local/lib/python3.14/site-packages/bluetti_mqtt/core/devices/ac500.py

# Add dc_input polling to AC500
RUN sed -i "s|ReadHoldingRegisters(70, 21),|ReadHoldingRegisters(70, 21),\n            ReadHoldingRegisters(170, 3),|" \
    /usr/local/lib/python3.14/site-packages/bluetti_mqtt/core/devices/ac500.py

# Create Python script to add MQTT config and publish
RUN cat > /patch_mqtt.py << 'PYSCRIPT'
import sys
filepath = '/usr/local/lib/python3.14/site-packages/bluetti_mqtt/mqtt_client.py'
with open(filepath, 'r') as f:
    content = f.read()

# 1. Add grid_charge_amps config
grid_charge_config = """    'grid_charge_amps': MqttFieldConfig(
        type=MqttFieldType.NUMERIC,
        setter=True,
        advanced=False,
        home_assistant_extra={
            'name': 'Grid Charge Amps',
            'step': 1,
            'min': 0,
            'max': 10,
            'unit_of_measurement': 'A',
        }
    ),"""

content = content.replace(
    """    'battery_range_end': MqttFieldConfig(
        type=MqttFieldType.NUMERIC,
        setter=True,
        advanced=False,
        home_assistant_extra={
            'name': 'Battery Range End',
            'step': 1,
            'min': 0,
            'max': 100,
            'unit_of_measurement': '%',
        }
    ),""",
    """    'battery_range_end': MqttFieldConfig(
        type=MqttFieldType.NUMERIC,
        setter=True,
        advanced=False,
        home_assistant_extra={
            'name': 'Battery Range End',
            'step': 1,
            'min': 0,
            'max': 100,
            'unit_of_measurement': '%',
        }
    ),
""" + grid_charge_config
)

# 2. Add dc_input configs
dc_input_config = """    'dc_input_voltage2': MqttFieldConfig(
        type=MqttFieldType.NUMERIC,
        setter=False,
        advanced=False,
        home_assistant_extra={
            'name': 'DC Input Voltage 2',
            'unit_of_measurement': 'V',
            'device_class': 'voltage',
            'state_class': 'measurement',
            'force_update': True,
        }
    ),
    'dc_input_power2': MqttFieldConfig(
        type=MqttFieldType.NUMERIC,
        setter=False,
        advanced=False,
        home_assistant_extra={
            'name': 'DC Input Power 2',
            'unit_of_measurement': 'W',
            'device_class': 'power',
            'state_class': 'measurement',
            'force_update': True,
        }
    ),"""

content = content.replace(
    """    'dc_input_current1': MqttFieldConfig(
        type=MqttFieldType.NUMERIC,
        setter=False,
        advanced=False,
        home_assistant_extra={
            'name': 'DC Input Current 1',
            'unit_of_measurement': 'A',
            'device_class': 'current',
            'state_class': 'measurement',
            'force_update': True,
        }
    ),""",
    """    'dc_input_current1': MqttFieldConfig(
        type=MqttFieldType.NUMERIC,
        setter=False,
        advanced=False,
        home_assistant_extra={
            'name': 'DC Input Current 1',
            'unit_of_measurement': 'A',
            'device_class': 'current',
            'state_class': 'measurement',
            'force_update': True,
        }
    ),
""" + dc_input_config
)

# 3. Add publish for dc_input_voltage2
content = content.replace(
    """        if 'internal_dc_input_voltage' in msg.parsed:
            await client.publish(
                topic_prefix + 'dc_input_voltage1',
                payload=str(msg.parsed['internal_dc_input_voltage']).encode()
            )""",
    """        if 'internal_dc_input_voltage' in msg.parsed:
            await client.publish(
                topic_prefix + 'dc_input_voltage1',
                payload=str(msg.parsed['internal_dc_input_voltage']).encode()
            )
        if 'dc_input_voltage2' in msg.parsed:
            await client.publish(
                topic_prefix + 'dc_input_voltage2',
                payload=str(msg.parsed['dc_input_voltage2']).encode()
            )"""
)

# 4. Add publish for dc_input_power2
content = content.replace(
    """        if 'internal_dc_input_power' in msg.parsed:
            await client.publish(
                topic_prefix + 'dc_input_power1',
                payload=str(msg.parsed['internal_dc_input_power']).encode()
            )""",
    """        if 'internal_dc_input_power' in msg.parsed:
            await client.publish(
                topic_prefix + 'dc_input_power1',
                payload=str(msg.parsed['internal_dc_input_power']).encode()
            )
        if 'dc_input_power2' in msg.parsed:
            await client.publish(
                topic_prefix + 'dc_input_power2',
                payload=str(msg.parsed['dc_input_power2']).encode()
            )"""
)

with open(filepath, 'w') as f:
    f.write(content)
    
print("✓ MQTT patches applied successfully")
PYSCRIPT

RUN python3 /patch_mqtt.py

CMD [ "/run.sh" ]