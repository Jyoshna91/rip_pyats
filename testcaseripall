from genie.testbed import load
from pyats.topology import loader
from pyats import aetest
import re, logging
import pdb 
from pyats.results import Passed,Failed
from pyats.async_.exceptions import *
from testcase_rip_lib import Rip_common_functions
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)
import time
global device_list

#easypy job_file -t testbed_yaml_file -datafile datafile

class CommonSetup(aetest.CommonSetup):

    @aetest.subsection
    def print_testbed_information(self,testbed):
        #pdb.set_trace()
        global uut1,uut2,uut3,device_list,device_info
        uut1 = testbed.devices['dut1']
        self.parent.parameters.update(uut1 = uut1)
        uut2 = testbed.devices['dut3']
        self.parent.parameters.update(uut2 = uut2)
        uut3 = testbed.devices['dut2']
        self.parent.parameters.update(uut3 = uut3)

        device_list = [uut1,uut2,uut3]
        device_info = {}

        if not testbed:
            logging.info("No testbed was provided to script launch")
        else:
            uut = testbed.devices['dut1']
            for device in testbed:
                logging.info("Device name : %s "%device.name)
                device_info.update({device.name: []})
                for intf in device:
                   logging.info("Interface : %s"%intf.name)
                   device_info[device.name].append(intf.name)
                   if intf.link:
                       logging.info("Link : %s"%intf.link.name)
                   else:
                       logging.info("Link : None")
            logger.info("Device and interfaces used for rip feature")
            logger.info(device_info)

    @aetest.subsection
    def connect_to_devices(self,testbed):
        logger.info("Connecting to devices")
        for uut in device_list:
            uut.connect()
            if uut.is_connected() == True:
                logging.info("Successfully connected to device %s"%uut.name)
                output = uut.execute('show version')
                res = Rip_common_functions.sh_version(output)
                logging.info("Software version :%s"%res['version'])
                logging.info("Image File :%s"%res['image'])
            else:
                logging.info("Device %s not connected"%uut.name)

    @aetest.subsection
    def configure_ip_address_to_interfaces(self,testbed):
        logger.info("Assign ip address to interfaces")
        #pdb.set_trace()
        logger.info(device_info.keys())
        #for dev in device_info.keys():
        Rip_common_functions.configure_ip_address(uut1,device1['intf'],device1['ip_address'],subnet_mask)
        Rip_common_functions.configure_ip_address(uut1,device1['intf1'],device1['ip_address1'],subnet_mask)
        Rip_common_functions.configure_ip_address(uut2,device2['intf1'],device2['ip_address1'],subnet_mask)
        Rip_common_functions.configure_ip_address(uut2,device2['intf2'],device2['ip_address2'],subnet_mask)
        Rip_common_functions.configure_ip_address(uut3,device3['intf'],device3['ip_address'],subnet_mask)
        Rip_common_functions.configure_ip_address(uut3,device3['intf1'],device3['ip_address1'],subnet_mask)
    
    @aetest.subsection
    def enabling_rip_on_devices(self,testbed):
        logger.info("Enabling rip on devices")
        Rip_common_functions.enabling_rip(uut1)
        Rip_common_functions.enabling_rip(uut2)
        Rip_common_functions.enabling_rip(uut3)

#    @aetest.subsection
#    def configure_rip_on_devices(self,testbed):
#        logger.info("Configure rip on interfaces")
#        acl_common_functions.configure_rip(uut1,device1['intf'])
#        acl_common_functions.configure_rip(uut2,device2['intf1'])
#        acl_common_functions.configure_rip(uut2,device2['intf2'])
#        acl_common_functions.configure_rip(uut3,device3['intf'])

#@aetest.skip("testing first scenario")
class Basic_Rip_testcase(aetest.Testcase):
    
    @aetest.setup
    def configure_rip_routers(self,testbed):

        logger.info("Configure rip on routers")
        Rip_common_functions.configure_rip_routers(uut1)
        Rip_common_functions.configure_rip_routers(uut2)
        Rip_common_functions.configure_rip_routers(uut3)

        logger.info("Configure rip in interface")
        Rip_common_functions.configure_rip_interface(uut1,device1['intf'])
        Rip_common_functions.configure_rip_interface(uut1,device1['intf1'])
        Rip_common_functions.configure_rip_interface(uut2,device2['intf1'])
        Rip_common_functions.configure_rip_interface(uut2,device2['intf2'])
        Rip_common_functions.configure_rip_interface(uut3,device3['intf'])
        Rip_common_functions.configure_rip_interface(uut3,device3['intf1'])

    @aetest.test
    def check_ping_after_rip_config(self,testbed):
        logger.info("Check rip configured or not ")
        rip_config = uut2.execute("show ip rip neighbor")
        logger.info(rip_config)
        if "rip-2" in rip_config:
            logger.info("RIP configured on device")
        else:
            self.errored('RIP is not configured on device')

        logger.info("Ping the ip configured on device2: {}".format(uut3.name))
        for i in range(5):
            result = uut2.execute("ping {}".format(device3['ip_address']))
        res_dict = Rip_common_functions.validate_ping(result)
        logger.info("++++++++++++++++++++++++++++")
        logger.info(res_dict)
        logger.info("++++++++++++++++++++++++++++")

        if ((res_dict['sent_pkt'] == res_dict['receive_pkt']) and (res_dict['pkt_loss'] == '0.00%')):
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt'],res_dict['pkt_loss']))
            self.passed("Success: After applied for Rip, ping got successful")
        else:
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt']
,res_dict['pkt_loss']))
            self.failed("Failed: After applied for Rip, ping got failed")


    @aetest.cleanup
    def unconfigure_rip_on_device(self,testbed):
        logger.info("Unconfigure rip in interface")
        Rip_common_functions.unconfigure_rip_interface(uut1,device1['intf'])
        Rip_common_functions.unconfigure_rip_interface(uut2,device2['intf1'])
        Rip_common_functions.unconfigure_rip_interface(uut2,device2['intf2'])
        Rip_common_functions.unconfigure_rip_interface(uut3,device3['intf'])

        logger.info("Unconfigure rip on routers")
        Rip_common_functions.unconfigure_rip_routers(uut1)
        Rip_common_functions.unconfigure_rip_routers(uut2)
        Rip_common_functions.unconfigure_rip_routers(uut3)

#@aetest.skip("testing second scenario")
class Rip_metric_offset(aetest.Testcase):

    @aetest.setup
    def configure_rip_on_device(self,testbed):

        logger.info("Configure rip on routers")
        Rip_common_functions.configure_rip_routers(uut1)
        Rip_common_functions.configure_rip_routers(uut2)
        Rip_common_functions.configure_rip_routers(uut3)

        logger.info("Configure rip in interface")
        Rip_common_functions.configure_rip_interface(uut1,device1['intf'])
        Rip_common_functions.configure_rip_interface(uut1,device1['intf1'])
        Rip_common_functions.configure_rip_interface(uut2,device2['intf1'])
        Rip_common_functions.configure_rip_interface(uut2,device2['intf2'])
        Rip_common_functions.configure_rip_interface(uut3,device3['intf'])
        Rip_common_functions.configure_rip_interface(uut3,device3['intf1'])

        logger.info("Configure rip metric-offset in interface")
        Rip_common_functions.configure_rip_metric_offset(uut1,device1['intf1'])
        Rip_common_functions.configure_rip_metric_offset(uut3,device3['intf1'])

    @aetest.test
    def check_ping_after_rip_metric_offset(self,testbed):

        logger.info("Check rip metric-offset configured or not ")
        rip_config = uut1.execute("show ip rip interface ethernet 1/11")
        logger.info(rip_config)
        if "metric 3" in rip_config:
            logger.info("Rip metric offset configured on device sucessfully")
        else:
            self.errored('Rip metric offset is not configured on device')

        logger.info("Ping the ip configured on device2: {}".format(uut3.name))
        for i in range(3):
            result = uut1.execute("ping {}".format(device3['ip_address']))
        res_dict = Rip_common_functions.validate_ping(result)
        logger.info("++++++++++++++++++++++++++++")
        logger.info(res_dict)
        logger.info("++++++++++++++++++++++++++++")

        if ((res_dict['sent_pkt'] == res_dict['receive_pkt']) and (res_dict['pkt_loss'] == '0.00%')):
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt'],res_dict['pkt_loss']))
            self.passed("Success: After applied rip metric offset ping got successful")
        else:
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt']
,res_dict['pkt_loss']))
            self.failed("Failed: After applied rip metric offset ping got failed")

    @aetest.cleanup
    def unconfigure_rip_metric_offset_on_device(self,testbed):
        logger.info("Unconfigure rip metric offset in interface")
        Rip_common_functions.unconfigure_rip_metric_offset(uut1,device1['intf1'])
        Rip_common_functions.unconfigure_rip_metric_offset(uut3,device3['intf1'])


#@aetest.skip("testing third scenario")
class Rip_Passive_Interface(aetest.Testcase):

    @aetest.setup
    def configure_acl_on_device(self,testbed):

        logger.info("Configure rip  passive interface")
        Rip_common_functions.configure_rip_passive_interface(uut1,device1['intf1'])
        Rip_common_functions.configure_rip_passive_interface(uut3,device3['intf1'])

    @aetest.test
    def Verify_ping_after_rip_passive_interface(self,testbed):

        logger.info("Check passive interface configured or not ")
        rip_config = uut1.execute("show ip rip interface eth1/11")
        logger.info(rip_config)
        if "passive" in rip_config:
            logger.info("Passive interface configured on device")
        else:
            self.errored('Passive interface is not configured on device')

        logger.info("Check Rip neighbor after configuring passive interface ")
        rip_config = uut1.execute("show ip rip neighbor")
        logger.info(rip_config)
        if "12.12.12.2" in rip_config:
            logger.info("Rip neighbor verification successful")
        else:
            self.errored('Rip neighbor verification unsucessful')

        logger.info("Ping the ip configured on device3 and device1: {} {}".format(uut1.name,uut3.name))
        for i in range(5):
            result = uut1.execute("ping {}".format(device3['ip_address']))
        res_dict = Rip_common_functions.validate_ping(result)
        logger.info("++++++++++++++++++++++++++++")
        logger.info(res_dict)
        logger.info("++++++++++++++++++++++++++++")


        if ((res_dict['sent_pkt'] == res_dict['receive_pkt']) and (res_dict['pkt_loss'] == '0.00%')):
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt'],res_dict['pkt_loss']))
            self.passed("Success: After passive interface ping got successful")
        else:
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt']
,res_dict['pkt_loss']))
            self.failed("Failed: After passive interface for Rip ping got failed")        


    @aetest.cleanup
    def unconfigure_rip_passive_interface(self,testbed):
        logger.info("Unconfigure Rip passive interface")
        Rip_common_functions.unconfigure_rip_passive_interface(uut1,device1['intf1'])
        Rip_common_functions.unconfigure_rip_passive_interface(uut3,device3['intf1']) 


#@aetest.skip('testing fourth testcase')
class Rip_timers(aetest.Testcase):

    @aetest.setup
    def configuring_rip_timers(self,testbed):

        logger.info("Configure Rip timers on routers")
        Rip_common_functions.configure_rip_timers(uut1)
        Rip_common_functions.configure_rip_timers(uut3)

    @aetest.test
    def check_ping_rip_timers(self,testbed):

        logger.info("Check Rip Timers configured or not ")
        rip_config = uut1.execute("show ip rip")
        logger.info(rip_config)
        if "Updates every 10 sec, expire in 60 sec" in rip_config:
            logger.info("Rip timers configured on device")
        else:
            self.errored('Rip timers are not configured on device')

        logger.info("Check Rip Timers configured or not ")
        rip_config1 = uut3.execute("show ip rip")
        logger.info(rip_config1)
        if "Updates every 10 sec, expire in 60 sec" in rip_config1:
            logger.info("Rip timers configured on device")
        else:
            self.errored('Rip timers are not configured on device')

        logger.info("Verifying Rip neighborship after configuring timers")
        rip_config1 = uut1.execute("show ip rip neighbor")
        logger.info(rip_config1)
        if "12.12.12.2" in rip_config1:
            logger.info("Rip neighborship successful")
        else:
            self.errored('Rip neighborship unsuccessful')


        logger.info("Ping the ip configured on device2: {} to check in bound".format(uut3.name))
        for i in range(3):
            result1 = uut2.execute("ping {}".format(device3['ip_address']))
        res_dict = Rip_common_functions.validate_ping(result1)
        logger.info("++++++++++++++++++++++++++++")
        logger.info(res_dict)
        logger.info("++++++++++++++++++++++++++++")

        if ((res_dict['sent_pkt'] == res_dict['receive_pkt']) and (res_dict['pkt_loss'] == '0.00%')):
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt'],res_dict['pkt_loss']))
            self.passed("Success: After applied for Rip ping got successful")
        else:
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt']
,res_dict['pkt_loss']))
            self.failed("Failed: After applied for Rip ping got failed")

    @aetest.cleanup
    def unconfigure_rip_timers_on_device(self,testbed):
        logger.info("Unconfigure Rip timers on Routers")
        Rip_common_functions.unconfigure_rip_timers(uut1)
        Rip_common_functions.unconfigure_rip_timers(uut3)

            

#@aetest.skip("testing fifth scenario")
class Rip_distance(aetest.Testcase):

    @aetest.setup
    def configure_rip_distance_on_device(self,testbed):

        logger.info("Configure rip distance on device1:{}".format(uut1.name))
        Rip_common_functions.configure_rip_distance(uut1)

        logger.info("Configure rip distance on device3:{}".format(uut3.name))
        Rip_common_functions.configure_rip_distance(uut3)

    @aetest.test
    def check_rip_distance_configured(self,testbed):
        logger.info("Check rip distance configured or not ")
        rip_config = uut1.execute("show ip rip")
        logger.info(rip_config)
        
        if "Admin-distance: 100" in rip_config:
            logger.info("Rip distance configured on device")
        else:
            self.errored('Rip distance is not configured on device')

        logger.info("Check Rip neighbor after configuring passive interface ")
        rip_config = uut1.execute("show ip rip neighbor")
        logger.info(rip_config)
        if "12.12.12.2" in rip_config:
            logger.info("Rip neighbor verification successful")
        else:
            self.errored('Rip neighbor verification unsucessful')

        logger.info("Ping the ip configured on device3 and device1: {} {}".format(uut1.name,uut3.name))
        for i in range(5):
            result = uut1.execute("ping {}".format(device3['ip_address']))
        res_dict = Rip_common_functions.validate_ping(result)
        logger.info("++++++++++++++++++++++++++++")
        logger.info(res_dict)
        logger.info("++++++++++++++++++++++++++++")

        if ((res_dict['sent_pkt'] == res_dict['receive_pkt']) and (res_dict['pkt_loss'] == '0.00%')):
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt'],res_dict['pkt_loss']))
            self.passed("Success: After rip distance ping got successful")
        else:
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt']
,res_dict['pkt_loss']))
            self.failed("Failed: After rip distance for Rip ping got failed")


    @aetest.cleanup
    def unconfigure_rip_distance(self,testbed):
        logger.info("Unconfigure rip distance on device1:{}".format(uut1.name))
        Rip_common_functions.unconfigure_rip_distance(uut1)

        logger.info("Unconfigure rip distance on device3:{}".format(uut3.name))
        Rip_common_functions.unconfigure_rip_distance(uut3)  

#@aetest.skip("testing sixth scenario")
class Rip_Authentication(aetest.Testcase):

    @aetest.setup
    def configure_rip_authentication_on_device(self,testbed):

        logger.info("Configure rip authentication on device1:{}".format(uut1.name))
        Rip_common_functions.configure_rip_authentication(uut1,device1['intf1'])
        Rip_common_functions.configure_rip_authentication(uut3,device3['intf1'])

    @aetest.test
    def verification_of_rip_authentication(self,testbed):
        logger.info("Verifying rip authentication ")
        rip_config = uut1.execute("show ip rip interface {}".format(device1['intf1']))
        logger.info(rip_config)
        if "Authentication Mode: md5  Keychain: rip" in rip_config:
            logger.info("RIP authentication configured on device")
        else:
            self.errored('RIP authentication is not configured on device')

        logger.info("Ping the ip configured on device2: {}".format(uut3.name))
        for i in range(5):
            result = uut2.execute("ping {}".format(device3['ip_address']))
        res_dict = Rip_common_functions.validate_ping(result)
        logger.info("++++++++++++++++++++++++++++")
        logger.info(res_dict)
        logger.info("++++++++++++++++++++++++++++")

        if ((res_dict['sent_pkt'] == res_dict['receive_pkt']) and (res_dict['pkt_loss'] == '0.00%')):
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt'],res_dict['pkt_loss']))
            self.passed("Success: After applied for Rip ping got successful")
        else:
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt']
,res_dict['pkt_loss']))
            self.failed("Failed: After applied for Rip ping got failed")

    @aetest.cleanup
    def unconfigure_rip_pauthentication(self,testbed):
        logger.info("Unconfigure Rip authentication on devices")
        Rip_common_functions.unconfigure_rip_authentication(uut1,device1['intf1'])
        Rip_common_functions.unconfigure_rip_authentication(uut3,device3['intf1'])


#@aetest.skip('testing seventh testcase')
class Rip_maxpaths(aetest.Testcase):

    @aetest.setup
    def configuring_rip_maxpaths(self,testbed):

        logger.info("Configure Rip maxpaths on routers")
        Rip_common_functions.configure_rip_maxpaths(uut1)
        Rip_common_functions.configure_rip_maxpaths(uut3)

    @aetest.test
    def check_ping_rip_maxpaths(self,testbed):

        logger.info("Check Rip maxpaths configured or not ")
        rip_config = uut1.execute("show ip rip")
        logger.info(rip_config)
        if "Max-paths: 5" in rip_config:
            logger.info("Rip max-paths configured on device")
        else:
            self.errored('Rip max-paths are not configured on device')

        logger.info("Check Rip maxpaths configured or not")
        rip_config1 = uut3.execute("show ip rip")
        logger.info(rip_config1)
        if "Max-paths: 5" in rip_config1:
            logger.info("Rip maxpaths configured on device")
        else:
            self.errored('Rip maxpaths are not configured on device')

        logger.info("Verifying Rip routes after configuring maxpaths")
        rip_config1 = uut1.execute("show ip route")
        logger.info(rip_config1)
        if "120" in rip_config1:
            logger.info("Rip routes are present")
        else:
            self.errored('Rip routes are not present')

        logger.info("Ping the ip configured on device2: {} to check in bound".format(uut3.name))
        for i in range(3):
            result1 = uut2.execute("ping {}".format(device3['ip_address']))
        res_dict = Rip_common_functions.validate_ping(result1)
        logger.info("++++++++++++++++++++++++++++")
        logger.info(res_dict)
        logger.info("++++++++++++++++++++++++++++")

        if ((res_dict['sent_pkt'] == res_dict['receive_pkt']) and (res_dict['pkt_loss'] == '0.00%')):
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt'],res_dict['pkt_loss']))
            self.passed("Success: After applied for Rip ping got successful")
        else:
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt']
,res_dict['pkt_loss']))
            self.failed("Failed: After applied for Rip ping got failed")

    @aetest.cleanup
    def unconfigure_rip_maxpaths_on_device(self,testbed):
        logger.info("Unconfigure Rip maxpaths on Routers")
        Rip_common_functions.unconfigure_rip_maxpaths(uut1)
        Rip_common_functions.unconfigure_rip_maxpaths(uut3)



#@aetest.skip('testing eighth testcase')
class Rip_redistribution(aetest.Testcase):

    @aetest.setup
    def configuring_rip_redistribution(self,testbed):

        logger.info("Configure Rip redistribution on routers")
        Rip_common_functions.configure_rip_redistribution(uut1)
        Rip_common_functions.configure_rip_redistribution(uut3)

    @aetest.test
    def check_ping_after_rip_redistribution(self,testbed):

        logger.info("Check Rip redistribution configured or not ")
        rip_config = uut1.execute("show ip rip")
        logger.info(rip_config)
        if "ospf-1          policy rip" in rip_config:
            logger.info("Rip redistribution configured on device")
        else:
            self.errored('Rip redistribution are not configured on device')

        logger.info("Check Rip redistribution configured or not")
        rip_config1 = uut3.execute("show ip rip")
        logger.info(rip_config1)

        if "ospf-1          policy rip" in rip_config1:
            logger.info("Rip redistribution configured on device")
        else:
            self.errored('Rip redistribution are not configured on device')

        logger.info("Verifying Rip routes after configuring redistribution")
        rip_config1 = uut1.execute("show ip route")
        logger.info(rip_config1)
        if "120" in rip_config1:
            logger.info("Rip routes are present")
        else:
            self.errored('Rip routes are not present')

        logger.info("Ping the ip configured on device2: {} to check in bound".format(uut3.name))
        for i in range(3):
            result1 = uut2.execute("ping {}".format(device3['ip_address']))
        res_dict = Rip_common_functions.validate_ping(result1)
        logger.info("++++++++++++++++++++++++++++")
        logger.info(res_dict)
        logger.info("++++++++++++++++++++++++++++")


        if ((res_dict['sent_pkt'] == res_dict['receive_pkt']) and (res_dict['pkt_loss'] == '0.00%')):
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt'],res_dict['pkt_loss']))
            self.passed("Success: After applied for Rip ping got successful")
        else:
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt']
,res_dict['pkt_loss']))
            self.failed("Failed: After applied for Rip ping got failed")

    @aetest.cleanup
    def unconfigure_rip_redistribution_on_device(self,testbed):
        logger.info("Unconfigure Rip redistribution on Routers")
        Rip_common_functions.unconfigure_rip_redistribution(uut1)
        Rip_common_functions.unconfigure_rip_redistribution(uut3)



#@aetest.skip("testing ninth scenario")
class Unconfigure_and_configure_Rip_testcase(aetest.Testcase):

    @aetest.setup
    def unconfigure_rip_routers(self,testbed):

        logger.info("Unconfigure rip on routers")
        Rip_common_functions.unconfigure_rip_routers(uut1)
        Rip_common_functions.unconfigure_rip_routers(uut2)
        Rip_common_functions.unconfigure_rip_routers(uut3)

        logger.info("Unconfigure rip in interface")
        Rip_common_functions.unconfigure_rip_interface(uut1,device1['intf'])
        Rip_common_functions.unconfigure_rip_interface(uut1,device1['intf1'])
        Rip_common_functions.unconfigure_rip_interface(uut2,device2['intf1'])
        Rip_common_functions.unconfigure_rip_interface(uut2,device2['intf2'])
        Rip_common_functions.unconfigure_rip_interface(uut3,device3['intf'])
        Rip_common_functions.unconfigure_rip_interface(uut3,device3['intf1'])

    @aetest.test
    def configure_rip_on_routers(self,testbed):

        logger.info("Configure rip on routers")
        Rip_common_functions.configure_rip_routers(uut1)
        Rip_common_functions.configure_rip_routers(uut2)
        Rip_common_functions.configure_rip_routers(uut3)

        logger.info("Configure rip in interface")
        Rip_common_functions.configure_rip_interface(uut1,device1['intf'])
        Rip_common_functions.configure_rip_interface(uut1,device1['intf1'])
        Rip_common_functions.configure_rip_interface(uut2,device2['intf1'])
        Rip_common_functions.configure_rip_interface(uut2,device2['intf2'])
        Rip_common_functions.configure_rip_interface(uut3,device3['intf'])
        Rip_common_functions.configure_rip_interface(uut3,device3['intf1'])

        logger.info("Check rip configured or not ")

        rip_config = uut1.execute("show ip rip neighbor")
        logger.info(rip_config)
        if "rip-2" in rip_config:
            logger.info("RIP configured on device")
        else:
            self.errored('RIP is not configured on device') 

        rip_config1 = uut2.execute("show ip rip neighbor")
        logger.info(rip_config1)
        if "rip-2" in rip_config1:
            logger.info("RIP configured on device")
        else:
            self.errored('RIP is not configured on device')

        rip_config2 = uut3.execute("show ip rip neighbor")
        logger.info(rip_config2)
        if "rip-2" in rip_config2:
            logger.info("RIP configured on device")
        else:
            self.errored('RIP is not configured on device')

        rip_config3 = uut1.execute("show ip route")
        logger.info(rip_config3)
        if "120" in rip_config3:
            logger.info("RIP configured on device")
        else:
            self.errored('RIP is not configured on device')

        logger.info("Ping the ip configured on device2: {}".format(uut3.name))
        for i in range(5):
            result = uut2.execute("ping {}".format(device3['ip_address']))
        res_dict = Rip_common_functions.validate_ping(result)
        logger.info("++++++++++++++++++++++++++++")
        logger.info(res_dict)
        logger.info("++++++++++++++++++++++++++++")

        if ((res_dict['sent_pkt'] == res_dict['receive_pkt']) and (res_dict['pkt_loss'] == '0.00%')):
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt'],res_dict['pkt_loss']))
            self.passed("Success: After applied for Rip ping got successful")
        else:
            logger.info("Sent : {} packets and received: {} packets and packet loss: {}".format(res_dict['sent_pkt'],res_dict['receive_pkt']
,res_dict['pkt_loss']))
            self.failed("Failed: After applied for Rip ping got failed")

    @aetest.cleanup
    def unconfigure_rip_on_device(self,testbed):
        logger.info("Unconfigure rip in interface")
        Rip_common_functions.unconfigure_rip_interface(uut1,device1['intf'])
        Rip_common_functions.unconfigure_rip_interface(uut1,device1['intf1'])
        Rip_common_functions.unconfigure_rip_interface(uut2,device2['intf1'])
        Rip_common_functions.unconfigure_rip_interface(uut2,device2['intf2'])
        Rip_common_functions.unconfigure_rip_interface(uut3,device3['intf'])
        Rip_common_functions.unconfigure_rip_interface(uut3,device3['intf1'])

        logger.info("Unconfigure rip on routers")
        Rip_common_functions.unconfigure_rip_routers(uut1)
        Rip_common_functions.unconfigure_rip_routers(uut2)
        Rip_common_functions.unconfigure_rip_routers(uut3)


class CommonCleanup(aetest.CommonCleanup):

    @aetest.subsection
    def unconfigure_rip_on_devices(self,testbed):
        logger.info("Unconfigure rip on interfaces")
        Rip_common_functions.disabling_rip(uut1)
        Rip_common_functions.disabling_rip(uut2)
        Rip_common_functions.disabling_rip(uut3)

    @aetest.subsection
    def unconfigure_ipaddress_device(self,testbed):
        logging.info("Unconfig ip address on interfaces of all devices")
        Rip_common_functions.unconfigure_ip_address(uut1,device1['intf'])
        Rip_common_functions.unconfigure_ip_address(uut1,device1['intf1'])
        Rip_common_functions.unconfigure_ip_address(uut2,device2['intf1'])
        Rip_common_functions.unconfigure_ip_address(uut2,device2['intf2'])
        Rip_common_functions.unconfigure_ip_address(uut3,device3['intf'])
        Rip_common_functions.unconfigure_ip_address(uut3,device3['intf1'])
 
    @aetest.subsection
    def disconnect(self,testbed):
        logger.info("Disconnect the devices")
        for uut in device_list:
            uut.disconnect()

 
if __name__ == '__main__':
    import argparse
    from pyats.topology import loader
 
    parser = argparse.ArgumentParser()
    parser.add_argument('--testbed', dest = 'testbed',
                        type = loader.load)
 
    args, unknown = parser.parse_known_args()
 
    aetest.main(**vars(args))
