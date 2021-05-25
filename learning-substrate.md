#  服务端/客户端
use std::io::{Error, Read, Write};
#使用标准读取
use std::net::{TcpListener, TcpStream};
#使用标准TCP协议
use std::thread;
#使用标准线路
use std::time;
#使用标准时间

#定义一个客户函数
fn handle_client(mut stream: TcpStream) -> Result<(), Error>{
    let mut buf = [0; 512];
    for _ in 0..1000 {
        let bytes_read = stream.read(&mut buf)?;
        if bytes_read == 0 {
            return Ok(());
        }

        stream.write(&buf[..bytes_read])?;
        thread::sleep(time::Duration::from_secs(1 as u64));
    }

    Ok(())
}

#定义一个主函数
fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    let mut thread_vec: Vec<thread::JoinHandle<()>> = Vec::new();

    for stream in listener.incoming() {
        let stream = stream.expect("failed!");
        let handle = thread::spawn(move || {
            handle_client(stream)
        .unwrap_or_else(|error| eprintln!("{:?}", error));
        });

        thread_vec.push(handle);
    }

    for handle in thread_vec {
        handle.join().unwrap();
    }

    Ok(())
}


#客户端

#使用标准RufReader
use std::io::{self, prelude::*, BufReader, Write};
#使用标准网络
use std::net::TcpStream;
use std::str;

#定义一个主函数用rust实现客户端
fn main() -> std::io::Result<()> {
    let mut stream = TcpStream::connect("127.0.0.1:8080")?;
    for _ in 0..10 {
        let mut input = String::new();
        io::stdin()
            .read_line(&mut input)
            .expect("Failed to read from stdin");
        stream
            .write(input.as_bytes())
            .expect("Failed to write to stream");
        
        let mut reader = BufReader::new(&stream);
        let mut buffer: Vec<u8> = Vec::new();
        reader
            .read_until(b'\n', &mut buffer)
            .expect("Could not read into buffer");
        println!("{}", 
            str::from_utf8(&buffer).expect("Could not write buffer as string"));
        println!("");
    }
    Ok(())
}

