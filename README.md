Cisco::IronPort(3)    User Contributed Perl Documentation   Cisco::IronPort(3)



NNAAMMEE
       Cisco::IronPort - Interface to Cisco IronPort Reporting API

SSYYNNOOPPSSIISS
               use Cisco::IronPort;

               my $ironport = Cisco::IronPort->new(
                       username => $username,
                       password => $password,
                       server   => $server
               );

               my %stats = $ironport->incoming_mail_summary_current_hour;

               print "Total Attempted Messages : $stats{total_attempted_messages}{count}\n";
               print "Clean Messages : $stats{clean_messages}{count} ($stats{clean_messages}{percent}%)\n";

               # prints...
               # Total Attempted Messages : 932784938
               # Clean Messages : (34%)

               # Print the destination domain and number of pending messages for any domain
               # with > 50 pending messages

               foreach my $domain ( keys %stats ) {
                       print "There are $stats{$domain}{active_recipients} pending message for $domain\n"
                               if ( $stats{$domain}{active_recipients} > 50
                                       and $stats{$domain}{latest_host_status} ne 'Down' )
               }

               # Call an alert() function for any users whom have sent mail that matched an outgoing content filter -
               # handy to warn of users whom have sent mail to a known phishing address.

               foreach my $user ( keys %users ) {
                       alert("$user had $users{$user}{outgoing_stopped_by_content_filter} matches for outgoing content filters")
                               if $users{$user}{outgoing_stopped_by_content_filter}
               }

MMEETTHHOODDSS
       _n_e_w _( _%_A_R_G_S _)

               my $ironport = Cisco::IronPort->new(
                       username => $username,
                       password => $password,
                       server   => $server
               );

       Creates a Cisco::IronPort object.  The constructor accepts a hash
       containing three mandatory and one optional parameter.

       username
          The username of a user authorised to access the reporting API.

       password
          The password of the username used to access the reporting API.

       server
          The target IronPort device hosting the reporting API.  This value
          must be either a resolvable hostname or an IP address.

       proto
          This optional parameter may be used to specify the protocol (either
          http or https) which should be used when connecting to the reporting
          API.  If unspecified this parameter defaults to https.

       _i_n_c_o_m_i_n_g___m_a_i_l___s_u_m_m_a_r_y___c_u_r_r_e_n_t___h_o_u_r

               my %stats = $ironport->incoming_mail_summary_current_hour;
               print "Total Attempted Messages : $stats{total_attempted_messages}{count}\n";

       Returns a nested hash containing incoming mail summary statistics for
       the current hourly period.  The hash has the structure show below:

               $stats = {
                 'statistic_name_1' => {
                   'count'   => $count,
                   'percent' => $percent
                 },
                 'statistic_name_2' => {
                   'count'   => $count,
                   'percent' => $percent
                 },
                 ...

                 'statistic_name_n => {
                   ...
                 }

       Valid statistic names are show below - these names are derived from
       those returned by the reporting API with all spaces converted to
       underscores and all characters lower-cased.

               stopped_by_reputation_filtering
               stopped_as_invalid_recipients
               stopped_by_content_filter
               total_attempted_messages
               total_threat_messages
               clean_messages
               virus_detected
               spam_detected

       _i_n_c_o_m_i_n_g___m_a_i_l___s_u_m_m_a_r_y___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash with the same structure and information as
       described above for the iinnccoommiinngg__mmaaiill__ssuummmmaarryy__ccuurrrreenntt__hhoouurr method, but
       for a time period covering the current day.

       _i_n_c_o_m_i_n_g___m_a_i_l___s_u_m_m_a_r_y___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing the incoming mail summary statistics for
       the current hour period unformated and as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data from the
       API call directly.

       _i_n_c_o_m_i_n_g___m_a_i_l___s_u_m_m_a_r_y___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing the incoming mail summary statistics for
       the current day period unformated and as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data from the
       API call directly.

       _i_n_c_o_m_i_n_g___m_a_i_l___d_e_t_a_i_l_s___c_u_r_r_e_n_t___h_o_u_r

               # Print a list of sending domains which have sent more than 50 messages
               # of which over 50% were detected as spam.

               my %stats = $ironport->incoming_mail_details_current_hour;

               foreach my $domain (keys %stats) {
                 if ( ( $stats{$domain}{total_attempted} > 50 ) and
                      ( int (($stats{$domain}{spam_detected}/$stats{$domain}{total_attempted})*100) > 50 ) {
                   print "Domain $domain sent $stats{$domain}{total_attempted} messages, $stats{$domain}{spam_detected} were marked as spam.\n"
                 }
               }

       Returns a nested hash containing details of incoming mail statistics
       for the current hour period.  The hash has the following structure:

               sending.domain1.com => {
                 begin_date                            => a human-readable timestamp at the beginning of the measurement interval (YYYY-MM-DD HH:MM TZ),
                 begin_timestamp                       => seconds since epoch at the beginning of the measurement interval (resolution of 100ms),
                 clean                                 => total number of clean messages sent by this domain,
                 connections_accepted                  => total number of connections accepted from this domain,
                 end_date                              => a human-readable timestamp at the end of the measurement interval (YYYY-MM-DD HH:MM TZ),
                 end_timestamp                         => seconds since epoch at the end of the measurement interval (resolution of 100ms),
                 orig_value                            => the domain name originally establishing the connection prior to any relaying or masquerading,
                 sender_domain                         => the sending domain,
                 spam_detected                         => the number of messages marked as spam from this domain,
                 stopped_as_invalid_recipients         => number of messages stopped from this domain due to invalid recipients,
                 stopped_by_content_filter             => number of messages stopped from this domain due to content filtering,
                 stopped_by_recipient_throttling       => number of messages stopped from this domain due to recipient throttling,
                 stopped_by_reputation_filtering       => number of messages stopped from this domain due to reputation filtering,
                 total_attempted                       => total number of messages sent from this domain,
                 total_threat                          => total number of messages marked as threat messages from this domain,
                 virus_detected                        => total number of messages marked as virus positive from this domain
               },
               sending.domain2.com => {
                 ...
               },
               ...
               sending.domainN.com => {
                 ...
               }

       Where each domain having sent email in the current hour period is used
       as the value of a hash key in the returned hash having the subkeys
       listed above.  For a busy device this hash may contain hundreds or
       thousands of domains so caution should be excercised in storing and
       parsing this structure.

       _i_n_c_o_m_i_n_g___m_a_i_l___d_e_t_a_i_l_s___c_u_r_r_e_n_t___d_a_y

       This method returns a nested hash as described in the
       iinnccoommiinngg__mmaaiill__ddeettaaiillss__ccuurrrreenntt__hhoouurr method above but for a period of the
       current day.  Consequently the returned hash may contain a far larger
       number of entries.

       _i_n_c_o_m_i_n_g___m_a_i_l___d_e_t_a_i_l_s___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing the incoming mail details for the current
       hour period as retrieved directly from the reporting API.  This method
       is useful is you wish to access and/or parse the results directly.

       _i_n_c_o_m_i_n_g___m_a_i_l___d_e_t_a_i_l_s___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing the incoming mail details for the current
       day period as retrieved directly from the reporting API.  This method
       is useful is you wish to access and/or parse the results directly.

       _t_o_p___u_s_e_r_s___b_y___c_l_e_a_n___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e_s___c_u_r_r_e_n_t___h_o_u_r

               # Print a list of our top internal users and number of messages sent.

               my %top_users = $ironport->top_users_by_clean_outgoing_messages_current_hour;

               foreach my $user (sort keys %top_users) {
                 print "$user - $top_users{clean_messages} messages\n";
               }

       Returns a nested hash containing details of the top ten internal users
       by number of clean outgoing messages sent for the current hour period.
       The hash has the following structure:

               'user1@domain.com' => {
                 begin_date            => a human-readable timestamp of the begining of the current hour period ('YYYY-MM-DD HH:MM TZ'),
                 begin_timestamp       => a timestamp of the beginning of the current hour period in seconds since epoch,
                 end_date              => a human-readable timestamp of the end of the current hour period ('YYYY-MM-DD HH:MM TZ'),
                 end_timestamp         => a timestamp of the end of the current hour period in seconds since epoch,
                 internal_user         => the email address of the user (this may also be 'unknown user' if the address cannot be determined),
                 clean_messages        => the number of clean messages sent by this user for the current hour period
               },
               'user2@domain.com' => {
                 ...
               },
               ...
               user10@domain.com' => {
                 ...
               }

       _t_o_p___u_s_e_r_s___b_y___c_l_e_a_n___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e_s___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing details of the top ten internal users
       by number of clean outgoing messages sent for the current day period.

       _t_o_p___u_s_e_r_s___b_y___c_l_e_a_n___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e_s___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing the details of the top ten internal users
       by number of clean outgoing messages sent for the current hour period
       as retrieved directly from the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_p___u_s_e_r_s___b_y___c_l_e_a_n___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e_s___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing the details of the top ten internal users
       by number of clean outgoing messages sent for the current day period as
       retrieved directly from the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _a_v_e_r_a_g_e___t_i_m_e___i_n___w_o_r_k_q_u_e_u_e___c_u_r_r_e_n_t___h_o_u_r

               my %stats = $ironport->average_time_in_workqueue_current_day;

               foreach my $i (sort keys %stats) {
                       print "$stats{$i}{end_date} : $stats{$i}{time}\n"
               }

               # Prints the average time a message spent in the workqueue for the current hourly period
               # e.g.
               # 2012-08-07 03:34 GMT : 1.76650943396
               # 2012-08-07 03:39 GMT : 4.97411003236
               # 2012-08-07 03:44 GMT : 0.955434782609
               # 2012-08-07 03:49 GMT : 3.38574040219
               # 2012-08-07 03:54 GMT : 2.32837301587
               # ...

       This method returns a nested hash containing statistics for the average
       time a message spent in the workqueue for the previous hourly period -
       the hash has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 time                  => the average time in seconds a message spent in the workqueue for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _a_v_e_r_a_g_e___t_i_m_e___i_n___w_o_r_k_q_u_e_u_e___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the average time a
       message spent in the workqueue for the previous daily period - the hash
       has the same structure as detailed in the
       aavveerraaggee__ttiimmee__iinn__wwoorrkkqquueeuuee__ccuurrrreenntt__hhoouurr above.

       _a_v_e_r_a_g_e___t_i_m_e___i_n___w_o_r_k_q_u_e_u_e___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the average time a message
       spent in the workqueue for the previous hourly period as retrieved
       directly from the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _a_v_e_r_a_g_e___t_i_m_e___i_n___w_o_r_k_q_u_e_u_e___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the average time a message
       spent in the workqueue for the previous daily period as retrieved
       directly from the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _a_v_e_r_a_g_e___i_n_c_o_m_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___h_o_u_r

               my %avg_msg_size = $ironport->average_incoming_message_size_current_hour;

               foreach my $mdata (sort keys %avg_msg_size) {
                       print "$avg_msg_size{$mdata}{end_date} : $avg_msg_size{$mdata}{message_size}\n";
               }

               # Prints the average incoming message size in bytes for the time sample periods in the previous hour.
               # e.g.
               # 2012-09-13 22:04 GMT : 111587.886555
               # 2012-09-13 22:09 GMT : 84148.6127168
               # 2012-09-13 22:14 GMT : 26486.8187919
               # 2012-09-13 22:19 GMT : 58772.1949153
               # ...

       This method returns a nested hash containing statistics for the average
       incoming message size in bytes for the previous hourly period - the
       hash has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 message_size          => the average incoming message size in bytes for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _a_v_e_r_a_g_e___i_n_c_o_m_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the average incoming message
       size in bytes for the previous hourly period as retrieved directly from
       the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _a_v_e_r_a_g_e___i_n_c_o_m_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the average incoming
       message size in bytes for the previous daily period - the hash has the
       same structure as detailed in the
       aavveerraaggee__iinnccoommiinngg__mmeessssaaggee__ssiizzee__ccuurrrreenntt__hhoouurr above.

       _a_v_e_r_a_g_e___i_n_c_o_m_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the average incoming message
       size in bytes for the previous daily period as retrieved directly from
       the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _a_v_e_r_a_g_e___m_e_s_s_a_g_e_s___i_n___w_o_r_k_q_u_e_u_e___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the average
       number of messages in the workqueue for the previous hourly period -
       the hash has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 messages              => the average number of messages in the workqueue for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _a_v_e_r_a_g_e___m_e_s_s_a_g_e_s___i_n___w_o_r_k_q_u_e_u_e___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the average number of
       messages in the workqueue for the previous hourly period as retrieved
       directly from the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _a_v_e_r_a_g_e___m_e_s_s_a_g_e_s___i_n___w_o_r_k_q_u_e_u_e___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the average number of
       messages in the workqueue for the previous daily period - the hash has
       the same structure as detailed in the
       aavveerraaggee__mmeessssaaggeess__iinn__wwoorrkkqquueeuuee__ccuurrrreenntt__hhoouurr above.

       _a_v_e_r_a_g_e___m_e_s_s_a_g_e_s___i_n___w_o_r_k_q_u_e_u_e___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the average number of
       messages in the workqueue for the previous daily period as retrieved
       directly from the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _a_v_e_r_a_g_e___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the average
       outgoing message size in bytes for the previous hourly period - the
       hash has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 message_size          => the average outgoing message size in bytes for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _a_v_e_r_a_g_e___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the average outgoing message
       size in bytes for the previous hourly period as retrieved directly from
       the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _a_v_e_r_a_g_e___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the average outgoing
       message size in bytes for the previous daily period - the hash has the
       same structure as detailed in the
       aavveerraaggee__oouuttooggiinngg__mmeessssaaggee__ssiizzee__ccuurrrreenntt__hhoouurr above.

       _a_v_e_r_a_g_e___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the average outgoing message
       size in bytes for the previous daily period as retrieved directly from
       the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _c_p_u___b_y___f_u_n_c_t_i_o_n___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the CPU
       usage by function for the previous hourly period - the hash has the
       following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 anti-spam             => the percentage of CPU time used for anti-spam functions,
                 anti-virus            => the percentage of CPU time used for anti-virus functions,
                 mail_processing       => the percentage of CPU time used for mail processing functions,
                 reporting             => the percentage of CPU time used for reporting functions,
                 quarantine            => the percentage of CPU time used for quarantine functions,
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _c_p_u___b_y___f_u_n_c_t_i_o_n___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the CPU usage by function
       for the previous hourly period as retrieved directly from the reporting
       API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _c_p_u___b_y___f_u_n_c_t_i_o_n___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the CPU usage by
       function for the previous daily period - the hash has the same
       structure as detailed in the ccppuu__bbyy__ffuunnccttiioonn__ccuurrrreenntt__hhoouurr above.

       _c_p_u___b_y___f_u_n_c_t_i_o_n___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the CPU usage by function
       for the previous daily period as retrieved directly from the reporting
       API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _i_n_c_o_m_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for incoming
       content filter matches for the previous hourly period - the hash has
       the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 content_filter        => the name of the content filter,
                 messages              => the number of incoming messages matched by the content filter in the previous hour period,
                 total_outgoing_matches=> the number of outgoing messages matched by the content filter in the previous hour period,
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _i_n_c_o_m_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for incoming content filter
       matches for the previous hourly period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _i_n_c_o_m_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for incoming content filter
       matches for the previous daily period - the hash has the same structure
       as detailed in the iinnccoommiinngg__ccoonntteenntt__ffiilltteerr__mmaattcchheess__ccuurrrreenntt__hhoouurr above.

       _i_n_c_o_m_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the incoming content filter
       matches for the previous daily period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _o_u_t_g_o_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for outgoing
       content filter matches for the previous hourly period - the hash has
       the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 content_filter        => the name of the content filter,
                 messages              => the number of outgoing messages matched by the content filter in the previous hour period,
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _o_u_t_g_o_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for outgoing content filter
       matches for the previous hourly period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _o_u_t_g_o_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for outgoing content filter
       matches for the previous daily period - the hash has the same structure
       as detailed in the oouuttggooiinngg__ccoonntteenntt__ffiilltteerr__mmaattcchheess__ccuurrrreenntt__hhoouurr above.

       _o_u_t_g_o_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the outgoing content filter
       matches for the previous daily period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _m_a_x_i_m_u_m___m_e_s_s_a_g_e_s___i_n___w_o_r_k_q_u_e_u_e___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the maximum
       number of messages in the workqueue for the previous hourly period -
       the hash has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 messages              => the maximum number of messages in the workqueue for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _m_a_x_i_m_u_m___m_e_s_s_a_g_e_s___i_n___w_o_r_k_q_u_e_u_e___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the maximum number of
       messages in the workqueue for the previous hourly period as retrieved
       directly from the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _m_a_x_i_m_u_m___m_e_s_s_a_g_e_s___i_n___w_o_r_k_q_u_e_u_e___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the maximum number of
       messages in the workqueue for the previous daily period - the hash has
       the same structure as detailed in the
       mmaaxxiimmuumm__mmeessssaaggeess__iinn__wwoorrkkqquueeuuee__ccuurrrreenntt__hhoouurr above.

       _m_a_x_i_m_u_m___m_e_s_s_a_g_e_s___i_n___w_o_r_k_q_u_e_u_e___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the maximum messages in the
       workqueue for the previous daily period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _m_e_m_o_r_y___p_a_g_e___s_w_a_p_p_i_n_g___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the number
       of memory pages swapped for the previous hourly period - the hash has
       the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 pages_swapped         => the number of memory pages swapped for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _m_e_m_o_r_y___p_a_g_e___s_w_a_p_p_i_n_g___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the number of memory pages
       swapped for the previous hourly period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _m_e_m_o_r_y___p_a_g_e___s_w_a_p_p_i_n_g___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the number of memory
       pages swapped for the previous daily period - the hash has the same
       structure as detailed in the mmeemmoorryy__ppaaggee__sswwaappppiinngg__ccuurrrreenntt__hhoouurr above.

       _m_e_m_o_r_y___p_a_g_e___s_w_a_p_p_i_n_g___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the number of memory pages
       swapped for the previous daily period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _o_v_e_r_a_l_l___c_p_u___u_s_a_g_e___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the overall
       CPU usage for the previous hourly period - the hash has the following
       structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 cpu_usage             => the total CPU usage for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _o_v_e_r_a_l_l___c_p_u___u_s_a_g_e___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the overall CPU usage for
       the previous hourly period as retrieved directly from the reporting
       API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _o_v_e_r_a_l_l___c_p_u___u_s_a_g_e___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the overall CPU usage
       for the previous daily period - the hash has the same structure as
       detailed in the oovveerraallll__ccppuu__uussaaggee__ccuurrrreenntt__hhoouurr above.

       _o_v_e_r_a_l_l___c_p_u___u_s_a_g_e___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the overall CPU usage for
       the previous daily period as retrieved directly from the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_p___i_n_c_o_m_i_n_g___v_i_r_u_s___t_y_p_e_s___d_e_t_e_c_t_e_d___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the top
       incoming virus types detected in the previous hourly period - the hash
       has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 messages              => the number of messages detected for the measurement period,
                 virus_type            => a comma-seperated list of the incoming virus types detected for this measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _t_o_p___i_n_c_o_m_i_n_g___v_i_r_u_s___t_y_p_e_s___d_e_t_e_c_t_e_d___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the top incoming virus types
       detected in the previous hourly period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_p___i_n_c_o_m_i_n_g___v_i_r_u_s___t_y_p_e_s___d_e_t_e_c_t_e_d___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the top incoming virus
       types detected in the previous daily period - the hash has the same
       structure as detailed in the
       ttoopp__iinnccoommiinngg__vviirruuss__ttyyppeess__ddeetteecctteedd__ccuurrrreenntt__hhoouurr above.

       _t_o_p___i_n_c_o_m_i_n_g___v_i_r_u_s___t_y_p_e_s___d_e_t_e_c_t_e_d___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the top incoming virus types
       detected for the previous daily period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_p___o_u_t_g_o_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the top
       outgoing content filter matches for the previous hourly period - the
       hash has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 content_filter        => the name of the content filter,
                 messages              => the number of outgoing messages matched by the content filter in the previous hour period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _t_o_p___o_u_t_g_o_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the top outgoing content
       content filter matches for the previous hourly period as retrieved
       directly from the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_p___o_u_t_g_o_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the top outgoing
       content filter matches for the previous daily period - the hash has the
       same structure as detailed in the
       ttoopp__oouuttggooiinngg__ccoonntteenntt__ffiilltteerr__mmaattcchheess__ccuurrrreenntt__hhoouurr above.

       _t_o_p___o_u_t_g_o_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a nested hash containing statistics for the top outgoing
       content filter matches for the previous daily period as retrieved
       directly from the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_p___i_n_c_o_m_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the top
       incoming content filter matches for the previous hourly period - the
       hash has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 content_filter        => the name of the content filter,
                 messages              => the number of incoming messages matched by the content filter in the previous hour period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _t_o_p___i_n_c_o_m_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the top incoming content
       content filter matches for the previous hourly period as retrieved
       directly from the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_p___i_n_c_o_m_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the top incoming
       content filter matches for the previous daily period - the hash has the
       same structure as detailed in the
       ttoopp__iinnccoommiinngg__ccoonntteenntt__ffiilltteerr__mmaattcchheess__ccuurrrreenntt__hhoouurr above.

       _t_o_p___i_n_c_o_m_i_n_g___c_o_n_t_e_n_t___f_i_l_t_e_r___m_a_t_c_h_e_s___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the top incoming content
       content filter matches for the previous daily period as retrieved
       directly from the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_t_a_l___i_n_c_o_m_i_n_g___c_o_n_n_e_c_t_i_o_n_s___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the total
       number of incoming connections for the previous hourly period - the
       hash has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 connections           => the total number of connections for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _t_o_t_a_l___i_n_c_o_m_i_n_g___c_o_n_n_e_c_t_i_o_n_s___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the total number of incoming
       connections for the previous hourly period as retrieved directly from
       the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_t_a_l___i_n_c_o_m_i_n_g___c_o_n_n_e_c_t_i_o_n_s___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the total number of
       incoming connections for the previous daily period - the hash has the
       same structure as detailed in the ttoottaall__iinnccoommiinngg
       ccoonnnneeccttiioonnss__ccuurrrreenntt__hhoouurr above.

       _t_o_t_a_l___i_n_c_o_m_i_n_g___c_o_n_n_e_c_t_i_o_n_s___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the total number of incoming
       connections for the previous daily period as retrieved directly from
       the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_t_a_l___i_n_c_o_m_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the total
       incoming message size in bytes for the previous hourly period - the
       hash has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 message_size          => the total incoming message size in bytes for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _t_o_t_a_l___i_n_c_o_m_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the total incoming message
       size in bytes for the previous hourly period as retrieved directly from
       the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_t_a_l___i_n_c_o_m_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the total incoming
       message size in bytes for the previous daily period - the hash has the
       same structure as detailed in the
       ttoottaall__iinnccoommiinngg__mmeessssaaggee__ssiizzee__ccuurrrreenntt__hhoouurr above.

       _t_o_t_a_l___i_n_c_o_m_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the total incoming message
       size in bytes for the previous daily period as retrieved directly from
       the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_t_a_l___i_n_c_o_m_i_n_g___m_e_s_s_a_g_e_s___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the total
       number of incoming messages for the previous hourly period - the hash
       has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 messages              => the total number of incoming messages for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _t_o_t_a_l___i_n_c_o_m_i_n_g___m_e_s_s_a_g_e_s___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the total number of incoming
       messages for the previous hourly period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_t_a_l___i_n_c_o_m_i_n_g___m_e_s_s_a_g_e_s___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the total number of
       incoming messages for the previous daily period - the hash has the same
       structure as detailed in the
       ttoottaall__nnuummbbeerr__ooff__iinnccoommiinngg__mmeessssaaggeess__ccuurrrreenntt__hhoouurr above.

       _t_o_t_a_l___i_n_c_o_m_i_n_g___m_e_s_s_a_g_e_s___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the total number of incoming
       messages for the previous daily period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_t_a_l___o_u_t_g_o_i_n_g___c_o_n_n_e_c_t_i_o_n_s___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the total
       number of outgoing connections for the previous hourly period - the
       hash has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 connections           => the total number of outgoing connections for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _t_o_t_a_l___o_u_t_g_o_i_n_g___c_o_n_n_e_c_t_i_o_n_s___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the total number of outgoing
       connections for the previous hourly period as retrieved directly from
       the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_t_a_l___o_u_t_g_o_i_n_g___c_o_n_n_e_c_t_i_o_n_s___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the total number of
       outgoing connections for the previous daily period - the hash has the
       same structure as detailed in the
       ttoottaall__nnuummbbeerr__oouuttggooiinngg__ccoonnnneeccttiioonnss__ccuurrrreenntt__hhoouurr above.

       _t_o_t_a_l___o_u_t_g_o_i_n_g___c_o_n_n_e_c_t_i_o_n_s___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the total number of outgoing
       connections for the previous daily period as retrieved directly from
       the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_t_a_l___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the total
       outgoing message size in bytes for the previous hourly period - the
       hash has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 message_size          => the total outgoing message size in bytes for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _t_o_t_a_l___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the total outgoing message
       size in bytes for the previous hourly period as retrieved directly from
       the reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_t_a_l___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the total outgoing
       message size in bytes for the previous daily period - the hash has the
       same structure as detailed in the
       ttoottaall__oouuttggooiinngg__mmeessssaaggee__ssiizzee__ccuurrrreenntt__hhoouurr above.

       _t_o_t_a_l___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e___s_i_z_e___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the total outgoing message
       size for the previous daily period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_t_a_l___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e_s___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing statistics for the total
       outgoing number of messages for the previous hourly period - the hash
       has the following structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 messages              => the total number of outgoing messages for the measurement period
               },
               measurement_period_2_begin_timestamp => {
                 ...
               },
               ...
               measurement_period_n_begin_timestamp => {
                 ...
               }

       _t_o_t_a_l___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e_s___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing statistics for the total number of outgoing
       messages for the previous hourly period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _t_o_t_a_l___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e_s___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing statistics for the total number of
       outgoing messages for the previous daily period - the hash has the same
       structure as detailed in the
       ttoottaall__nnuummbbeerr__ooff__oouuttggooiinngg__mmeessssaaggeess__ccuurrrreenntt__hhoouurr above.

       _t_o_t_a_l___o_u_t_g_o_i_n_g___m_e_s_s_a_g_e_s___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing statistics for the total number of outgoing
       messages for the previous daily period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _i_n_t_e_r_n_a_l___u_s_e_r___d_e_t_a_i_l_s___c_u_r_r_e_n_t___h_o_u_r

       This method returns a nested hash containing details of the mail sent
       by each internal user for the previous hourly period - the hash has the
       following structure:

               user1@domain.com => {
                       begin_date                              => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                       begin_timestamp                         => a timestamp marking the beginning of the measurement period in seconds since epoch,
                       end_date                                => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                       end_timestamp                           => a timestamp marking the ending of the measurement period in seconds since epoch,
                       incoming_clean                          => the number of incoming clean messages sent to this user,
                       incoming_content_filter_matches         => the number of incoming messages sent to this user matching ian incoming content filter,
                       incoming_spam_detected                  => the number of incoming messages sent to this user that were detected as spam,
                       incoming_stopped_by_content_filter      => the number of incoming messages sent to this user that were stopped by content filtering,
                       incoming_virus_detected                 => the number of incoming messages sent to this user that were detected as virii,
                       internal_user                           => the email address of the internal user,
                       outgoing_clean                          => the number of outgoing clean messages sent by this user,
                       outgoing_content_filter_matches         => the number of outgoing messages sent by this user matching outgoing content filters,
                       outgoing_spam_detected                  => the number of outgoing messages sent by this user that were detected as spam,
                       outgoing_stopped_by_content_filter      => the number of outgoing messages sent by this user stopped by outgoing content filters,
                       outgoing_virus_detected                 => the number of outgoing messages sent by this user matching outgoing virus filters
               },
               user2@domain.com => {
                       ...
               },
               ...
               userN@domain.com => {
                       ...
               }

       _i_n_t_e_r_n_a_l___u_s_e_r___d_e_t_a_i_l_s___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing details of the mail sent by each internal
       user for the previous hourly period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _i_n_t_e_r_n_a_l___u_s_e_r___d_e_t_a_i_l_s___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing details of the mail sent by each
       internal user for the previous daily period - the hash has the same
       structure as detailed in the iinntteerrnnaall__uusseerr__ddeettaaiillss__ccuurrrreenntt__hhoouurr above.

       _i_n_t_e_r_n_a_l___u_s_e_r___d_e_t_a_i_l_s___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing details of the mail sent by each internal
       user for the previous daily period as retrieved directly from the
       reporting API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _c_o_n_t_e_n_t___f_i_l_t_e_r___d_e_t_a_i_l___c_u_r_r_e_n_t___h_o_u_r _( _{ _f_i_l_t_e_r___n_a_m_e _=_> _$_f_i_l_t_e_r___n_a_m_e _} _)

       Given an anonymous hash containing the key 'filter_name' where the
       value is a valid content filter name defined on the target device, this
       method returns a nested hash containing details of the content filter
       statistics for the previous hourly period - the hash has the following
       structure:

               measurement_period_1_begin_timestamp => {
                 begin_timestamp       => a timestamp marking the beginning of the measurement period in seconds since epoch,
                 end_timestamp         => a timestamp marking the ending of the measurement period in seconds since epoch,
                 begin_date            => a human-readable timestamp marking the beginning of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 end_date              => a human-readable timestamp marking the ending of the measurement period (YYYY-MM-DD HH:MM:SS TZ),
                 key                   => the user-specified name of the content filter that was matched,
                 internal_user         => the email address of the internal user sending the email matching the content filter,
                 messages              => the total number of messages matching this content_filter for the measurement period
               },
               measurement_period_1_begin_timestamp => {

       _c_o_n_t_e_n_t___f_i_l_t_e_r___d_e_t_a_i_l___c_u_r_r_e_n_t___h_o_u_r___r_a_w _( _{ _f_i_l_t_e_r___n_a_m_e _=_> _$_f_i_l_t_e_r___n_a_m_e
       _} _)

       Returns a scalar containing details of the content filter statistics
       for the previous hourly period as retrieved directly from the reporting
       API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _c_o_n_t_e_n_t___f_i_l_t_e_r___d_e_t_a_i_l___c_u_r_r_e_n_t___d_a_y _( _{ _f_i_l_t_e_r___n_a_m_e _=_> _$_f_i_l_t_e_r___n_a_m_e _} _)

       Given an anonymous hash containing the key 'filter_name' where the
       value is a valid content filter name defined on the target device, this
       method returns a nested hash containing details of the content filter
       statistics for the previous daily period - the hash has the same
       structure as detailed in the ccoonntteenntt__ffiilltteerr__ddeettaaiill__ccuurrrreenntt__hhoouurr method.

       _c_o_n_t_e_n_t___f_i_l_t_e_r___d_e_t_a_i_l___c_u_r_r_e_n_t___d_a_y___r_a_w _( _{ _f_i_l_t_e_r___n_a_m_e _=_> _$_f_i_l_t_e_r___n_a_m_e _}
       _)

       Returns a scalar containing details of the content filter statistics
       for the previous daily period as retrieved directly from the reporting
       API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _o_u_t_g_o_i_n_g___d_e_l_i_v_e_r_y___s_t_a_t_u_s___c_u_r_r_e_n_t___h_o_u_r

       Returns a nested hash containing details of outgoing delivery status
       statistics for the previous hourly period indexed by destination domain
       - the hash has the following structure:

               domain-1.com => {
                 active_recipients     => the number of messages currently pending delivery for this domain,
                 connections_out       => the number of active outbound connections to this domain,
                 delivered recipients  => the number of messages delivered to this domain,
                 destination_domain    => the destination domain name (this is the same as the hash key),
                 hard_bounced          => the number of messages sent to this domain that were hard bounced,
                 latest_host_status    => the last recorded host status (e.g. up or down),
                 soft_bounced          => the number of messages sent to this domain that were soft bounced,
               },
               domain-2.com => {
                 ...
               },
               ...
               domain-n.com => {
                 ...
               }

       _o_u_t_g_o_i_n_g___d_e_l_i_v_e_r_y___s_t_a_t_u_s___c_u_r_r_e_n_t___h_o_u_r___r_a_w

       Returns a scalar containing details of outgoing delivery status
       statistics for the previous hourly period as returned directly from the
       API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

       _o_u_t_g_o_i_n_g___d_e_l_i_v_e_r_y___s_t_a_t_u_s___c_u_r_r_e_n_t___d_a_y

       Returns a nested hash containing details of outgoing delivery status
       statistics for the previous daily period indexed by destination domain
       - the hash has the same structure as the hash returned in the
       oouuttggooiinngg__ddeelliivveerryy__ssttaattuuss__ccuurrrreenntt__hhoouurr method.

       _o_u_t_g_o_i_n_g___d_e_l_i_v_e_r_y___s_t_a_t_u_s___c_u_r_r_e_n_t___d_a_y___r_a_w

       Returns a scalar containing details of outgoing delivery status
       statistics for the previous daily period as returned directly from the
       API.

       This method may be useful if you wish to process the raw data retrieved
       from the API yourself.

AAUUTTHHOORR
       Luke Poskitt, "<ltp at cpan.org>"

BBUUGGSS
       Please report any bugs or feature requests to "bug-cisco-ironport at
       rt.cpan.org", or through the web interface at
       http://rt.cpan.org/NoAuth/ReportBug.html?Queue=Cisco-IronPort
       <http://rt.cpan.org/NoAuth/ReportBug.html?Queue=Cisco-IronPort>.  I
       will be notified, and then you'll automatically be notified of progress
       on your bug as I make changes.

SSUUPPPPOORRTT
       You can find documentation for this module with the perldoc command.

           perldoc Cisco::IronPort

       You can also look for information at:

       ·   RT: CPAN's request tracker (report bugs here)

           http://rt.cpan.org/NoAuth/Bugs.html?Dist=Cisco-IronPort
           <http://rt.cpan.org/NoAuth/Bugs.html?Dist=Cisco-IronPort>

       ·   AnnoCPAN: Annotated CPAN documentation

           http://annocpan.org/dist/Cisco-IronPort
           <http://annocpan.org/dist/Cisco-IronPort>

       ·   CPAN Ratings

           http://cpanratings.perl.org/d/Cisco-IronPort
           <http://cpanratings.perl.org/d/Cisco-IronPort>

       ·   Search CPAN

           http://search.cpan.org/dist/Cisco-IronPort/
           <http://search.cpan.org/dist/Cisco-IronPort/>

LLIICCEENNSSEE AANNDD CCOOPPYYRRIIGGHHTT
       Copyright 2013 Luke Poskitt.

       This program is free software; you can redistribute it and/or modify it
       under the terms of either: the GNU General Public License as published
       by the Free Software Foundation; or the Artistic License.

       See http://dev.perl.org/licenses/ for more information.



perl v5.12.4                      2013-05-28                Cisco::IronPort(3)
