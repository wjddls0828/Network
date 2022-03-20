# Network

## route_output 함수가 필요한 이유
: ROUTING TABLE에서 DESTINATION IP와 longest matching되는 ENTRY를 찾기 위해 필요하다.  패킷을 destination으로 보내기 위해 해당되는 entry를 찾아서, 그 다음에 해당 entry의 next hop을 보고 거기로 forwarding을 해야하기 때문에 이 함수의 작업이 우선적으로 필요하다.

## 코드 설명
### 1) my_ip_output 함수
    
  - rentry에 next hop에 대한 정보가 들어있고, next hop이 NULL이면(즉, next hop이 destination IP, directly connected)라면 next_hop에 destination IP를 채워준다. 그렇지 않다면(else) rentry의 longest matching으로 라우팅 테이블에서 찾은 next hop에 있는 정보가 들어있으므로 그것을 next_hop에 넣어준다.
   
  - Cache_prt를 통해 ARP 캐시 엔트리 리스트들을 순회하여 ARP 테이블의 destination IP와 위에서 설정해둔 next_hop의 IP가 동일한 ENTRY를 찾는다. 찾았다면 해당 IP의 맥주소를 next_hop_mac에 저장하고 반복문에서 빠져나온다.
   
  - IP 헤더를 붙이는 과정에서 destination IP, source 길이, source, ttl을 설정한다. Destination ip를 destination의 길이만큼 buffer_ptr이 가르키는 곳에 메모리 카피를 한다. 카피한 후에 버퍼 포인터에 카피한 만큼의 크기를 더해서 뒤의 source len을 쓸 곳으로 포인터를 이동시킨다. source_len의 경우는 빅엔디안의 표기를 위해서 htons 함수를 통해 변환시킨 뒤에 buffer_ptr이 가르키는 곳에 넣고 또 다음의 source ip를 쓰기 위한 곳으로 buffer pointer를 이동시킨다. Source ip도 destination ip와 동일한 방식으로 진행하고 마지막에 ttl도 동일한 방식으로 설정해주고 버퍼 포인터를 이동시킨다.
  - 
### 2) my_ip_forward 함수
   
  - TTL을 체크하는 부분이다. 패킷이 라우터에 도달할때마다 TTL은 1씩 감소되고, 패킷을 받고 TTL을 감소시킨 뒤에 만약 TTL이 0이 되었으면 더 이상 포워딩하지 않고 DROP된다. DROP하기 위해서 return -1을 한다.
  
### 3) processEthernet 함수
 
- eh는 이더넷 헤더 구조체의 변수인데, 여기서 ethertype을 가져와서 빅엔디안 방식으로 변환하기 위해 htons 함수를 호출해서 ethtype에 저장한다. 만약 가져온 ethtype이 0xfffe라면 이더넷 헤더만큼 버퍼 포인터를 이동한 뒤에, 데이터 payload부분만 my_ip_receive에 넘겨준다. 즉 IP 단으로 넘겨준다. 
