import { NodeSSH } from 'node-ssh';
import { EventEmitter } from 'events';

interface SSHConfig {
  host: string;
  port: number;
  username: string;
  password?: string;
  privateKey?: string;
}

class SSHConnectionError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'SSHConnectionError';
  }
}

export class SSHManager extends EventEmitter {
  private ssh: NodeSSH;
  private config: SSHConfig;
  private isConnected: boolean = false;

  constructor(config: SSHConfig) {
    super();
    this.ssh = new NodeSSH();
    this.config = config;
  }

  /**
   * Establishes SSH connection using provided credentials
   * @throws SSHConnectionError if connection fails
   */
  public async connect(): Promise<void> {
    try {
      await this.ssh.connect({
        host: this.config.host,
        port: this.config.port,
        username: this.config.username,
        password: this.config.password,
        privateKey: this.config.privateKey,
        timeout: 10000
      });
      
      this.isConnected = true;
      this.emit('connected');
    } catch (error) {
      throw new SSHConnectionError(
        `Failed to connect: ${error instanceof Error ? error.message : 'Unknown error'}`
      );
    }
  }

  /**
   * Executes command on remote server
   * @param command Command to execute
   * @returns Promise with command output
   */
  public async executeCommand(command: string): Promise<string> {
    if (!this.isConnected) {
      throw new SSHConnectionError('Not connected to SSH server');
    }

    try {
      const result = await this.ssh.execCommand(command);
      if (result.stderr) {
        throw new Error(result.stderr);
      }
      return result.stdout;
    } catch (error) {
      throw new SSHConnectionError(
        `Command execution failed: ${error instanceof Error ? error.message : 'Unknown error'}`
      );
    }
  }

  /**
   * Closes SSH connection
   */
  public async disconnect(): Promise<void> {
    if (this.isConnected) {
      this.ssh.dispose();
      this.isConnected = false;
      this.emit('disconnected');
    }
  }

  /**
   * Validates SSH key format
   * @param key SSH public key to validate
   */
  public static validateSSHKey(key: string): boolean {
    const sshKeyRegex = /^(ssh-rsa|ssh-ed25519)\s+[A-Za-z0-9+/]+[=]{0,3}\s+.+$/;
    return sshKeyRegex.test(key.trim());
  }
}
# myconstantstore
members area page
