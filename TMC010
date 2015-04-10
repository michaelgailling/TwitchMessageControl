# TWITCH MESSAGE CONTROL
# Some code has been borrowed from https://www.sevadus.tv/forums/index.php?/topic/774-simple-python-irc-bot/
# Anything else belongs to me
# Michael Gailling
# v0.010 Basic Funtionality Testing

import socket

# Private login info
from twitchlogin import *                       # Strings within are called NICK and PASS
                                                # You will need these for twitch chat

# Server info
HOST = "irc.twitch.tv"                          # Hostname of the IRC-Server in this case twitch's
PORT = 6667                                     # Default IRC-Port
CHAN = "#somechannelname"                       # Channelname = #{Nickname}


def send_pong(msg):
    con.send(bytes('PONG %s\r\n' % msg, 'UTF-8'))


def send_message(chan, msg):
    con.send(bytes('PRIVMSG %s :%s\r\n' % (chan, msg), 'UTF-8'))


def send_nick(nick):
    con.send(bytes('NICK %s\r\n' % nick, 'UTF-8'))


def send_pass(password):
    con.send(bytes('PASS %s\r\n' % password, 'UTF-8'))


def join_channel(chan):
    con.send(bytes('JOIN %s\r\n' % chan, 'UTF-8'))


def part_channel(chan):
    con.send(bytes('PART %s\r\n' % chan, 'UTF-8'))


class MessageControl:
    def __init__(self):
        self.data_type = ''
        self.usr_name = ''
        self.data_str = ''
        self.content_size = 0
        self.content_list = []
        self.data_list = []
        self.name_list = []                 # Active names list do not reset this unless you have a good reason
        self.type_dict = {                  # IRC response type dictionary
            "001": "rpl_welcome",
            "002": "rpl_yourhost",
            "003": "rpl_created",
            "004": "rpl_myinfo",
            "353": "rpl_namereply",
            "366": "rpl_endofnames",
            "372": "rpl_motd",
            "375": "rpl_motdstart",
            "376": "rpl_endofmotd",
            "PING": "ping",
            "JOIN": "join",
            "PART": "part",
            "PRIVMSG": "privmsg",
            "MODE": "mode"
        }
        self.action_dict = {                # Dictionary of actions triggered by chat
            "!names": self.say_names,       # These are like directories for switch case statements
            "!love": self.say_love,         # But because python has it's own way we must do it that way
            "!rules": self.say_rules        # I almost like it better this way
        }

    # This function is for handling message headers
    def parse_msg(self, data_input):
        # --------------------------------------------------------------------
        # | Reset everything below before reading a new message              |
        # | These variables only hold their value for the life of the message|
        # | This for the sake of conserving memory in the long run           |
        # | Data will be later written another way                           |
        # --------------------------------------------------------------------
        self.data_type = ''
        self.usr_name = ''
        self.content_size = 0
        self.content_list = []
        self.data_list = []

        # If the end of line is detected...
        if "\r\n" in data_input:

            # Replace  : ! = from the raw string with spaces
            data_input = data_input.replace(":", " ").replace("!", " ", 1).replace("=", " ", 1)

            # Split the raw string, using spaces as seperators, into a list
            self.data_list = data_input.split()

            # Detect data type
            if len(self.data_list) < 3:
                for i in range(0, 2):
                    if self.data_list[i] in self.type_dict:
                        self.set_data_type(self.type_dict[self.data_list[i]])

            else:
                for i in range(0, 3):
                    if self.data_list[i] in self.type_dict:
                        self.set_data_type(self.type_dict[self.data_list[i]])

            if self.data_type == "ping":
                send_pong(self.data_list[1])

            # Set content and content size
            if self.data_type == "privmsg" or self.data_type == "rpl_namereply":
                self.set_content_list(self.data_list[4:])
                self.content_size = len(self.content_list)

            # Set MODE content
            elif self.data_type == "mode":
                self.set_content_list(self.data_list[3])

            # Set username
            if self.data_type == "privmsg" or self.data_type == "join" or self.data_type == "part":
                self.set_usr_name(self.data_list[0])

            # Set MODE username
            elif self.data_type == "mode":
                self.set_usr_name(self.data_list[4])

            return True  # Might change this as triggering from this function might be a bad idea

    def read_msg(self):
        if self.data_type == "rpl_namereply":
            for i in self.content_list:
                if i not in self.name_list:
                    self.name_list.append(i)

        elif self.data_type == "part":
            if self.usr_name in self.name_list:
                self.name_list.remove(self.usr_name)

        elif self.data_type == "join":
            if self.usr_name not in self.name_list:
                self.name_list.append(self.usr_name)

        elif self.data_type == "privmsg":
            if self.content_list[0] in self.action_dict:
                self.action_dict[self.content_list[0]]()

    def say_names(self):
        x = ""
        for i in self.name_list:
            x += i + " "
        send_message(CHAN, x)

    def say_love(self):
        send_message(CHAN, "Robot Loves You @" + self.usr_name)

    def say_rules(self):
        send_message(CHAN, "@" + self.usr_name + " I am the law!")

    def spew_console(self):
        # Print the result of data parse to console
        # This helps with debugging
        print(self.data_type)

        print(self.usr_name)

        print(self.content_size)

        print(self.content_list)

        print(self.data_list)

        print(self.name_list)

        print("\r\n")

    # Setters
    def set_data_type(self, data_type):
        self.data_type = data_type

    def set_usr_name(self, usr):
        self.usr_name = usr

    def set_content_list(self, content):
        self.content_list = content

    def set_content_size(self, size):
        self.content_size = size

#Initialize
msgctrl = MessageControl()
data = ""

# Connect 
con = socket.socket()
con.connect((HOST, PORT))
send_pass(PASS)
send_nick(NICK)
join_channel(CHAN)

# Main loop
while True:
    data = data + con.recv(1).decode('UTF-8', errors='ignore')
    if msgctrl.parse_msg(data):
        data = ""
        msgctrl.read_msg()
        msgctrl.spew_console()
