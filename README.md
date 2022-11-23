kafka-topics --bootstrap-server localhost:19092 --command-config configs/host.properties --list
kafka-topics --bootstrap-server localhost:19092 --command-config configs/host.properties --create --topic test
kafka-console-producer --bootstrap-server localhost:19092 --producer.config configs/host.properties --topic test
kafka-console-consumer --bootstrap-server localhost:19092 --consumer.config configs/host.properties --topic test