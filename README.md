change-charset-database
=======================

#!/usr/bin/php
<?php

  ini_set('default_charset','UTF-8');
	$host = "127.0.0.1";
    $user = "root";
    $password = "";
    $dbase = "database";

    mysql_connect($host,$user,$password) or die ('Erro ao conectar na base de dados');
    mysql_select_db($dbase);

    $tables = array();
    $result = mysql_query('SHOW TABLES');
    $tables = array();
    while($row = mysql_fetch_row($result)){
        $tables[] = $row[0];
    }
        
    foreach ($tables as $key => $value) {
        $primary = mysql_fetch_assoc(mysql_query("SHOW KEYS FROM ".$value." WHERE Key_name = 'PRIMARY'"));
        
        update(select($value), $value, $primary['Column_name']);
    }

    function select($table){
        $sql = "SELECT * FROM ".$table;
        mysql_set_charset('latin1');
        $result = mysql_query($sql);
        $retorno = array();
        while($row = mysql_fetch_assoc($result)){
            $retorno[] = $row;

        }
        return $retorno;
    }

    function update($result, $table, $primary){
                
        $update = 'UPDATE '.$table.' SET ';
        foreach ($result as $value) {
            foreach($value as $key=>$field){
                if($primary == $key){
                    $where = ' WHERE '.$primary.' = '.$field;
                } else {
                    $update .= $table.".".$key.' = ';         
                	if(mb_detect_encoding($field, 'utf8, latin1') == 'ISO-8859-1' )
                    {
                        $update .= !is_numeric($field) ? "'".mysql_escape_string(utf8_encode($field))."', " : $field.', ';

                    } else {
                        $update .= !is_numeric($field)? "'".mysql_escape_string($field)."', " : $field.', ';
                    }
                }
            }
            $update = substr($update,0,-2);
            $update .= $where;
            mysql_set_charset('utf8');
            if(!mysql_query($update)){
                die($update);
            }
            $update = 'UPDATE '.$table.' SET '; 
            $where = '';
        }
                               
    }

?>
