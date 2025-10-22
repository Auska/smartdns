# Add source files in a specific order to ensure proper linking
set(SMARTDNS_SOURCES
    # Core files
    "dns.c"
    "dns_cache.c"
    "dns_plugin.c"
    "dns_stats.c"
    "proxy.c"
    "smartdns.c"
    "timer.c"
    "tlog.c"
    
    # Lib files
    "lib/conf.c"
    "lib/idna.c"
    "lib/art.c"
    "lib/bitops.c"
    "lib/radix.c"
    "lib/rbtree.c"
    "lib/stringutil.c"
    "lib/timer_wheel.c"
    
    # Utils files
    "utils/capbility.c"
    "utils/daemon.c"
    "utils/dns_debug.c"
    "utils/ipset.c"
    "utils/misc.c"
    "utils/neighbors.c"
    "utils/net.c"
    "utils/nftset.c"
    "utils/ssl.c"
    "utils/stack.c"
    "utils/tls_header_parse.c"
    "utils/url.c"
    
    # DNS client files
    "dns_client/client_http3.c"
    "dns_client/client_https.c"
    "dns_client/client_mdns.c"
    "dns_client/client_quic.c"
    "dns_client/client_socket.c"
    "dns_client/client_tcp.c"
    "dns_client/client_tls.c"
    "dns_client/client_udp.c"
    "dns_client/conn_stream.c"
    "dns_client/dns_client.c"
    "dns_client/ecs.c"
    "dns_client/group.c"
    "dns_client/packet.c"
    "dns_client/pending_server.c"
    "dns_client/proxy.c"
    "dns_client/query.c"
    "dns_client/server_info.c"
    "dns_client/wake_event.c"
    
    # DNS conf files
    "dns_conf/address.c"
    "dns_conf/bind.c"
    "dns_conf/bootstrap_dns.c"
    "dns_conf/client_rule.c"
    "dns_conf/client_subnet.c"
    "dns_conf/cname.c"
    "dns_conf/conf_file.c"
    "dns_conf/ddns_domain.c"
    "dns_conf/dhcp_lease_dnsmasq.c"
    "dns_conf/dns_conf.c"
    "dns_conf/dns_conf_group.c"
    "dns_conf/dns64.c"
    "dns_conf/domain_rule.c"
    "dns_conf/domain_set.c"
    "dns_conf/get_domain.c"
    "dns_conf/group.c"
    "dns_conf/host_file.c"
    "dns_conf/https_record.c"
    "dns_conf/ip_alias.c"
    "dns_conf/ip_rule.c"
    "dns_conf/ip_set.c"
    "dns_conf/ipset.c"
    "dns_conf/local_domain.c"
    "dns_conf/nameserver.c"
    "dns_conf/nftset.c"
    "dns_conf/plugin.c"
    "dns_conf/proxy_names.c"
    "dns_conf/proxy_server.c"
    "dns_conf/ptr.c"
    "dns_conf/qtype_soa.c"
    "dns_conf/server.c"
    "dns_conf/server_group.c"
    "dns_conf/set_file.c"
    "dns_conf/smartdns_domain.c"
    "dns_conf/speed_check_mode.c"
    "dns_conf/srv_record.c"
    
    # DNS server files
    "dns_server/address.c"
    "dns_server/answer.c"
    "dns_server/audit.c"
    "dns_server/cache.c"
    "dns_server/client_rule.c"
    "dns_server/cname.c"
    "dns_server/connection.c"
    "dns_server/context.c"
    "dns_server/dns_server.c"
    "dns_server/dualstack.c"
    "dns_server/ip_rule.c"
    "dns_server/ipset_nftset.c"
    "dns_server/local_addr.c"
    "dns_server/mdns.c"
    "dns_server/neighbor.c"
    "dns_server/prefetch.c"
    "dns_server/ptr.c"
    "dns_server/request.c"
    "dns_server/request_pending.c"
    "dns_server/rules.c"
    "dns_server/server_https.c"
    "dns_server/server_socket.c"
    "dns_server/server_tcp.c"
    "dns_server/server_tls.c"
    "dns_server/server_udp.c"
    "dns_server/speed_check.c"
    "dns_server/soa.c"
    
    # Fast ping files
    "fast_ping/fast_ping.c"
    "fast_ping/notify_event.c"
    "fast_ping/ping_fake.c"
    "fast_ping/ping_host.c"
    "fast_ping/ping_icmp.c"
    "fast_ping/ping_icmp6.c"
    "fast_ping/ping_tcp.c"
    "fast_ping/ping_udp.c"
    "fast_ping/wakeup_event.c"
    
    # HTTP parse files
    "http_parse/http_parse.c"
    "http_parse/http1_parse.c"
    "http_parse/http2_parse.c"
    "http_parse/http3_parse.c"
    "http_parse/qpack.c"
)

# Create static library
add_library(smartdns_lib STATIC ${SMARTDNS_SOURCES})

# Set include directories for the library
target_include_directories(smartdns_lib PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)

# Create main executable
add_executable(smartdns main.c)

# Link the library with the executable
target_link_libraries(smartdns smartdns_lib)

# Set compile definitions for the library
target_compile_definitions(smartdns_lib PRIVATE
    BASE_FILE_NAME="main.c"
)

# Set compile definitions for the executable
target_compile_definitions(smartdns PRIVATE
    BASE_FILE_NAME="main.c"
)

# Link libraries
if(STATIC)
    target_link_libraries(smartdns
        ssl
        crypto
        -Wl,--whole-archive
        pthread
        -Wl,--no-whole-archive
        dl
        m
        ${ATOMIC_LIBRARIES}
        -static
        -rdynamic
    )
else()
    target_link_libraries(smartdns
        ssl
        crypto
        pthread
        dl
        m
        ${ATOMIC_LIBRARIES}
        -rdynamic
    )
endif()

# Install the binary
install(TARGETS smartdns RUNTIME DESTINATION /usr/sbin)
